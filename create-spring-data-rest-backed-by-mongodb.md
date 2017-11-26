# Create app

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

Create the Author entity

`./src/main/java/com/joaovicente/observablespring/Author.java`

```
package com.joaovicente.observablespring;

import lombok.Data;
import org.springframework.data.annotation.Id;

@Data
public class Author {
    @Id private String id;
    private String email;
    private String name;
}
```

And its repository

...

---



```
package com.joaovicente.author;
import lombok.Data;
import org.springframework.data.annotation.Id;

@Data
public class Author {
    @Id private String id;
    @Version Long version // ETag support
    private String name;
    private String email;
}
```

`./src/main/java/com/joaovicente/author/Author.java`

```
package com.joaovicente.author;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AuthorApplication {

    public static void main(String[] args) {
        SpringApplication.run(AuthorApplication.class, args);
    }
```

`./src/main/java/com/joaovicente/author/AuthorRepository.java`

```
package com.joaovicente.author;

import java.util.List;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.repository.query.Param;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

@RepositoryRestResource(collectionResourceRel = "authors", path = "authors")
public interface AuthorRepository extends MongoRepository<Author, String> {
    List<Author> findByName(@Param("name") String name);
}
```

```
mvn spring-boot:run
```

# Docker setup

Follow Dockerising... steps to build a docker image

When done, add the following to ./src/main/resources/application.properties

```
spring.data.mongodb.host=mongoserver
spring.data.mongodb.port=27017
```

and rebuild the docker container:

```
mvn clean package docker:build
```

Create Docker compose `./docker-compose.yml`

```
author:
  image: joaovicente/author:latest
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

Now start-up the mongodb and author containers

```
docker-compose.yml
```

When you POST an Author

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

Load test using Taurus \([https://hub.docker.com/r/blazemeter/taurus/\](https://hub.docker.com/r/blazemeter/taurus/%29\)

```
sudo pip install bzt
```

create a `load-test.yml` file

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

WIP script \(not failing but not creating data\)

```
---
execution:
- concurrency: 1
  hold-for: 10s
  scenario: author-create
  #write-xml-jtl: full

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



