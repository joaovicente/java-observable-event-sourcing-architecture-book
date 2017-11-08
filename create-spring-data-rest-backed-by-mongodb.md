Assumes MongoDB is already running

```
spring init \
    -d=data-rest,data-mongodb,lombok \
    -groupId=com.joaovicente \
    -artifactId=author \
    -name=author \
    author
```

`./src/main/java/com/joaovicente/author/Author.java`

```
package com.joaovicente.author;
import lombok.Data;
import org.springframework.data.annotation.Id;

@Data
public class Author {
    @Id private String id;
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



