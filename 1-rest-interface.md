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

The handler method as shown below, to expose`POST /authors`

```java
package com.joaovicente.stories;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMethod;

@RestController
public class CreateAuthorController {
    @RequestMapping(value = "/authors", method = RequestMethod.POST)

    public CreateAuthorDto postAuthor(@RequestBody CreateAuthorDto createAuthorDto) {
        return createAuthorDto;
    }
}
```

which will use the`/src/main/java/com/joaovicente/CreateAuthorDto.java` shown below

```java
package com.joaovicente.stories;

@lombok.Data
public class CreateAuthorDto {
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

I am going to use [httpie](https://httpie.org/) instead of curl to interact with the REST interface

So here goes a POST /authors

```
http POST localhost:8080/authors name=joao email=joao.diogo.vicente@gmail.com
```

and the output shows the DTO returned as expected

```
HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Date: Sat, 13 Jan 2018 23:00:26 GMT
Transfer-Encoding: chunked

{
    "email": "joao.diogo.vicente@gmail.com", 
    "name": "joao"
}
```



