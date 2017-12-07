# Create Spring Boot app

From the command line execute

```
spring init \        
    -d=data-rest,data-mongodb,lombok,data-rest-hal \
    -groupId=com.joaovicente \
    -artifactId=observablespring \
    -name=observablespring \
    -bootVersion=2.0.0.M6 \    
    observablespring
```

# Be Docker ready

## Build Docker image using a Maven docker plugin

Go into your project directory

```
cd observablespring
```

And edit the `pom.xml` adding

in properties section

```
    <properties>
        ...
        <docker.image.prefix>joaovicente</docker.image.prefix>
    </properties>
```

And in the plugins section add the following

```
        <plugins>
               ...
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>0.4.11</version>
                <configuration>
                    <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                    <imageTags>
                        <imageTag>${project.version}</imageTag>
                        <imageTag>latest</imageTag>
                    </imageTags>
                    <baseImage>frolvlad/alpine-oraclejdk8:slim</baseImage>
                    <entryPoint>["java", "-jar", "/${project.build.finalName}.jar"]</entryPoint>

                    <!--<dockerDirectory>src/main/docker</dockerDirectory>-->
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
        </plugins>
```

## Create a Docker compose file to to run both the app and MongoDB

create `docker-compose.yml`

```
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
```

## Create Entities and Repositories

## Author

Create the Author entity

`./src/main/java/com/joaovicente/observablespring/Author.java`

```java
package com.joaovicente.observablespring;

import org.springframework.data.annotation.Id;
import lombok.Data;

@Data
public class Author {
    @Id private String id;
    private String email;
    private String name;
}
```

And its repository

`./src/main/java/com/joaovicente/observablespring/AuthorRepository.java`

```java
package com.joaovicente.observablespring;

import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.repository.query.Param;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;
import java.util.List;


@RepositoryRestResource(collectionResourceRel = "authors", path = "authors")
public interface AuthorRepository extends MongoRepository<Author, String> {
    List<Author> findByName(@Param("name") String name);
}
```

## Story

Create the Story entity

`./src/main/java/com/joaovicente/observablespring/Story.java`

```java
package com.joaovicente.observablespring;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.DBRef;
import lombok.Data;

@Data
public class Story {
    @Id private String id;
    private String title;
    private String body;
    @DBRef
    Author author;
}
```

And its repository

`./src/main/java/com/joaovicente/observablespring/StoryRepository.java`

```java
package com.joaovicente.observablespring;

import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.repository.query.Param;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

import java.util.List;

@RepositoryRestResource(collectionResourceRel = "stories", path = "stories")
public interface StoryRepository extends MongoRepository<Story, String> {
    List<Story> findByTitle(@Param("title") String title);
}
```

# Configure MongoDB

add mongodb connection settings to `./src/main/resources/application.properties`

```
spring.data.mongodb.host=mongodb
spring.data.mongodb.port=27017
```

# Package the app as a docker image

```
mvn clean package docker:build
```

# Running the app with a MongoDB backend, using Docker Compose

```
docker-compose up
```

# Explore the REST API

## Create an Author

```
curl -X POST \
  http://localhost:8080/authors \
  -H 'content-type: application/json' \
  -d '{
  "name":"Joao",
  "email":"joao.diogo.vicente@gmail.com"
}'
```

You will see the Author created

```
{
  "name" : "Joao",
  "email" : "joao.diogo.vicente@gmail.com",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/authors/5a038cab52faff0001e77123"
    },
    "author" : {
      "href" : "http://localhost:8080/authors/5a038cab52faff0001e77123"
    }
  }
}
```

If you follow the URL shown in `_links.self.href`

```
curl http://localhost:8080/authors/5a038cab52faff0001e77123
```

you should be able to GET the Author

```
{
  "name" : "Joao",
  "email" : "joao.diogo.vicente@gmail.com",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/authors/5a038cab52faff0001e77123"
    },
    "author" : {
      "href" : "http://localhost:8080/authors/5a038cab52faff0001e77123"
    }
  }
}
```

## Create a Story

Create a story linked to the Author above

```
curl -X POST \
  http://localhost:8080/stories \
  -H 'content-type: application/json' \
  -d '{
    "title":"Once",
    "body":"Once upon a time ...",
    "author":"/authors/5a038cab52faff0001e77123"
}'
```

which will return

```
{
    "title": "Once",
    "body": "Once upon a time ...",
    "_links": {
        "self": {
            "href": "http://localhost:8080/stories/5a29c3ec52faff0001ee98e9"
        },
        "story": {
            "href": "http://localhost:8080/stories/5a29c3ec52faff0001ee98e9"
        },
        "author": {
            "href": "http://localhost:8080/stories/5a29c3ec52faff0001ee98e9/author"
        }
    }
}
```

## Inspect MongoDB

```
docker exec -it observablespring_mongodb_1 sh

#mongo

>db.story.find()
{ "_id" : ObjectId("5a29c3ec52faff0001ee98e9"), "_class" : "com.joaovicente.observablespring.Story", "title" : "Once", "body" : "Once upon a time ...", "author" : DBRef("author", ObjectId("5a038cab52faff0001e77123")) }
```

---

## Load test

Taurus \([https://hub.docker.com/r/blazemeter/taurus/](https://hub.docker.com/r/blazemeter/taurus/%29%29\) is a very useful load test framework. We'll use it to put some load through the service

```
sudo pip install bzt
```

create a simple`load-test.yml` file

```
---
execution:
- concurrency: 10
  ramp-up: 1m
  hold-for: 1m30s
  scenario: simple

scenarios:
  simple:
    think-time: 0.75
    requests:
    - http://localhost:8080/authors
```

Now execute the load

```
bzt load-test.yml
```

And you should see a nice ASCII dashboard showing how the Author service is coping with the load

Now lets create a more interesting test `author-create-load-test.yml`which will actually create users

```
---
execution:
- concurrency: 1
  hold-for: 10s
  scenario: author-create
  write-xml-jtl: full

scenarios:
  author-create:
    think-time: 1
    data-sources:
      - author-create.csv
    requests:
    - url: http://localhost:8080/authors
      method: POST
      headers:
        Content-Type: application/json
      body:
        name: ${name}
        email: ${email}
```

sourced from `author-create.csv`

```
name,email
antoinette,antoinette@gmail.com
brian,brian@yahoo.com
carl,carl@gmail.com
david,david@yahoo.com
edith,edith@gmail.com
frank,frank@yahoo.com
greg,greg@gmail.com
hugo,hugo@yahoo.com
isabel,isabel@gmail.com
john,john@yahoo.com
kevin,kevin@gmail.com
lidia,lidia@yahoo.com
mark,mark@yahoo.com
```

Run it the same way

```
bzt author-create-load-test.yml
```

And it will have created 10 users, 1 per second, which is not much of a load test but it illustrates how to create one

If you want to do more on the load test side the [Taurus JMeter manual](https://github.com/Blazemeter/taurus/blob/master/site/dat/docs/JMeter.md) should help. You will find good info about using data sources in the [JMeter examples](https://github.com/Blazemeter/taurus/tree/master/examples/jmeter).

