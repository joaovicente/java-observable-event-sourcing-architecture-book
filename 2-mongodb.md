# MongoDB

In this chapter we are going to persist the Author we received in the previous chapter.

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

```
package com.joaovicente.stories;

import org.springframework.data.mongodb.repository.MongoRepository;

public interface AuthorRepository extends MongoRepository<Author, String> {
}


```



