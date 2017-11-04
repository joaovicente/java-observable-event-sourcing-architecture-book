## Enabling Swagger

Add springfox dependencies to the `pom.xml`

```
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.6.1</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.6.1</version>
</dependency>
```

Create a swagger config class in `./src/main/java/com/joaovicente/SwaggerConfig.java`

```java
com.joaovicente.author;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.regex("/users.*"))
//                .paths(PathSelectors.any())
                .build();
    }
}
```

After a rebuild you should be able to see the Swagger JSON at: 

http://localhost:8080/v2/api-docs 

and see the swagger-ui at: 

[http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)





