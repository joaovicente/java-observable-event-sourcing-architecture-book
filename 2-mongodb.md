# MongoDB

In this chapter we are going to persist the Author we received in the previous chapter using MongoDB

> Reference: [Spring MongoDB quickstart](https://spring.io/guides/gs/accessing-data-mongodb/)

## Setup MongoDB

> TODO: MongoDB setup \(provide docker instructions\)

## Carry on coding

So, firstly we create the Author entity class `./src/main/java/com/joaovicente/CreateAuthorController.java`

```java
package com.joaovicente.stories;

import lombok.Builder;
import lombok.Data;
import org.springframework.data.annotation.Id;

@Builder
@Data
public class Author {
    @Id
    String id;
    private String name;
    private String email;
}
```

And the Author repository

```java
package com.joaovicente.stories;

import org.springframework.data.mongodb.repository.MongoRepository;

public interface AuthorRepository extends MongoRepository<Author, String> {
}
```

Now let's modify the `CreateAuthorController` class POST request handler to construct an `Author` object from the `CreateAuthorDto` and persist it using the `AuthorRepository`

```java
package com.joaovicente.stories;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMethod;


@RestController
public class CreateAuthorController {
    @Autowired
    private AuthorRepository repository;
    @RequestMapping(value = "/authors", method = RequestMethod.POST)

    public Author createAuthor(@RequestBody CreateAuthorDto createAuthorDto) {
        Author author = Author.builder()
                .name(createAuthorDto.getName())
                .email(createAuthorDto.getEmail()).build();
        repository.insert(author);
        return author;
    }
}
```

Now when we POST /authors

```
http POST localhost:8080/authors name=joao email=joao.diogo.vicente@gmail.com
```

We are also returned an id

```
http POST localhost:8080/authors name=diogo email=diogo.vicente@gmail.com 

HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Date: Sat, 13 Jan 2018 23:36:32 GMT
Transfer-Encoding: chunked

{
    "email": "joao.diogo.vicente@gmail.com", 
    "id": "5a5a980076f4641204dfe0c8", 
    "name": "joao"
}
```

and the Author is now persisted in MongoDB `author` collection

