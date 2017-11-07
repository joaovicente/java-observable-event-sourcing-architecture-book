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



