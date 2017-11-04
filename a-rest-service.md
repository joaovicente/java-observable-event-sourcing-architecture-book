# The Authors REST service

Let's start by building the Author service

```bash
spring init \ 
-d=web \
-groupId=com.joaovicente \
-artifactId=author \
-name=author \
author
```

Let's build the app and compile it

```
cd author
mvn clean package
```

And run it

```
mvn spring-boot:run
```

So the application runs but is does not do anything useful, so lets stop the app now with Ctrl-C and let's create a `AuthorController`by editing`./src/main/java/com/joaovicente/AuthorController.java`

Now create the`AuthorController`class and annotate it as a`@RestController`and add the`@RequestMapping`handler method as shown below, to expose the `GET /authors/{authorId}` endpoint

```java
package com.joaovicente.author;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PathVariable;
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
    public String postAuthor(@RequestBody Author author)   {    
        String resp = "Author: " + author.getName() + "," + author.getEmail();
        return resp;
    }
}
```

```bash
mvn spring-boot:run
```

When you curl the endpoint

```bash
curl http://localhost:8080/authors/123
```

You will now see the `Nothing yet for author 123` response

