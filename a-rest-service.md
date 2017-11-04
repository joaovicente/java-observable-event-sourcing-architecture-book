# The Authors REST service

Let's start by building the Author service

```
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

```
package com.joaovicente.author;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AuthorController {
    @RequestMapping(value="/authors/{authorId}")
    public String author(String authorId)   {
        return "No authors yet!\n";
    }
}
```

Run the app again

```
mvn spring-boot:run
```

When you curl the endpoint

`curl http://localhost:8080/authors/123`

You will now see the `No authors yet!` pre-baked response

