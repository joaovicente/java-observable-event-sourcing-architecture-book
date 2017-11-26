[https://reflectoring.io/documenting-spring-data-rest-api-with-springfox/](https://reflectoring.io/documenting-spring-data-rest-api-with-springfox/) is a good guide for Spring data Swagger support

A summary of steps is shown beloe

In pom.xml you will need the following dependencies

```
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.7.0</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-data-rest</artifactId>
            <version>2.7.0</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.7.0</version>
        </dependency>
```

And

On your application class you will need `@EnableSwagger2` and `@Import(SpringDataRestConfiguration.class)`

```
package com.joaovicente.author;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Import;
import springfox.documentation.spring.data.rest.configuration.SpringDataRestConfiguration;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@SpringBootApplication
@EnableSwagger2
@Import(SpringDataRestConfiguration.class)
public class AuthorApplication {

    public static void main(String[] args) {
        SpringApplication.run(AuthorApplication.class, args);
    }
}
```

and you will need to define `SpringFoxConfiguration.java`, in this case exposing `/authors` and `/stories` paths

```
import com.google.common.base.Predicate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import static springfox.documentation.builders.PathSelectors.regex;
import static com.google.common.base.Predicates.*;

@Configuration
public class SpringFoxConfiguration {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                    .paths(paths())
                    .build();
    }

    private Predicate<String> paths() {
        return or(
                regex("/authors.*"),
                regex("/stories.*"));
    }
}
```



