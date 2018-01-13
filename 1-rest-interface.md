# 1 REST service

Let's start by building the Author service. We'll depend on `web` for REST capabilities and `lombok` to auto-generate getters and setters for the REST DTOs. We're also going to throw in `data-mongodb` and `kafka` dependencies as we are going to use then in the next 2 chapters.

```bash
spring init \
    -d=web,lombok,data-mongodb,kafka \
    -groupId=com.joaovicente \
    -artifactId=stories \
    -name=stories \
    stories
```

Let's compile the spring boot app

```
cd stories
mvn clean package
```

And run it

```
mvn spring-boot:run
```

So the application runs but is does not do anything useful, so lets stop the app now and let's create a`CreatAuthorController`by editing`./src/main/java/com/joaovicente/CreateAuthorController.java`and add  POST capabilities

The handler method as shown below, to expose the`GET /authors/{authorId}`endpoint and the `POST /authors`

```java
package com.joaovicente.author;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

@RestController
@RequestMapping(value="/authors")
public class AuthorController {
    @GetMapping(value="/{id}")
    public String getAuthor(@PathVariable String id) {
        String resp = "Nothing yet for author " + id + "\n";
        return resp;
    }
    @PostMapping()
    public String postAuthor(@RequestBody AuthorDto author)   {    
        String resp = author.getName() + "," + author.getEmail() + "\n";
        return resp;
    }
```

which will use the`/src/main/java/com/joaovicente/AuthorDto.java` shown below

```java
package com.joaovicente.author;

@lombok.Data
public class AuthorDto {
    private String name;
    private String email;
}
```

Let's run the app

```bash
mvn spring-boot:run
```

and try out the `GET` endpoint

```bash
curl http://localhost:8080/authors/123
```

You will now see the `Nothing yet for author 123` response

The  `POST` curl command below

```
curl -X POST \
  http://localhost:8080/authors \
  -H 'content-type: application/json' \
  -d '{
  "name":"Joao",
  "email":"joao.diogo.vicente@gmail.com"
}'
```

returns the expected a the concatenated values passed in via the `POST` request `Joao,joao.diogo.vicente@gmail.com`

