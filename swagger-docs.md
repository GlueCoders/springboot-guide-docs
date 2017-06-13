## Documenting REST Services using Swagger

Swagger(OpenAPI) has become the defacto standard to document the REST APIs. For this project springfox is used which plays nicely with Spring MVC. Let's get started.  
Code reference: [https://github.com/GlueCoders/springboot-guide/releases/tag/swagger-docs](https://github.com/GlueCoders/springboot-guide/releases/tag/swagger-docs)  

### POM Dependency
```
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>2.7.0</version>
</dependency>
```  

### Enabling Swagger in Application  
To enable Swagger in application a Docket bean has to be created with some settings. Even with default settings basic swagger works fine. Below is the configuration class which provides the Docket bean.  
```
package org.gluecoders.library.config;

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
    public Docket swaggerSpringMvcPlugin() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any())
                        .paths(PathSelectors.any())
                        .build();
    }
}
```

This configuration is going to select all controllers that are out there in our application (Books.class for example) and it will create a swagger json for those controllers.

### Enable Swagger UI
To see swagger docs, swagger-ui webjar is also needed. Let's include that also in pom file as shown below.  
```
<dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger-ui</artifactId>
   <version>2.7.0</version>
</dependency>
```  

Now start the server and visit [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html), Swagger UI should be up and running showing the Books controller method and REST endpoints.

[Prev](/logging.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](#)
