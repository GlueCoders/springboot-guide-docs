## Hello World Example
Code reference : [https://github.com/GlueCoders/springboot-guide/releases/tag/quickhelloworld](https://github.com/GlueCoders/springboot-guide/releases/tag/quickhelloworld)  
Let's add a simple webservice method in the project, to see some action. First include `spring-boot-starter-web` dependency as shown below.  
```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
```  
This will bring in a lot of dependencies, of which major are related to `spring-web`, `tomcat`. Now let's create a class which we will the start point of the application and will also serve as our webservice entry point.  
```
package org.gluecoders.library;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @RequestMapping("/")
    @GetMapping
    public String hello(){
        return "Hi ! Welcome to Spring Boot Guide";
    }
}
```  
Now run this class from IDE, Spring Boot logs should appear in console.

On browser hit [http://localhost:8080/](http://localhost:8080/), and a response will appear in return with text "Hi ! Welcome to Spring Boot Guide". 

Let's start one by one with the details going on in class
- `@SpringBootApplication` - Under the hood it enables a lot of annotations which are needed to enable Spring to be able to integrate with project like `ComponentScan`, `Configuration` etc.  
- `@RequestController` - Notifies Spring that this class is to be mapped to incoming HTTP requests.  
- `Main method` - This is the entry point when project is started. This delegates control to `SpringApplication` which in turn will start embedded `Tomcat` container and bootstrap code and other Spring related libraries together.  
- `@RequestMapping @GetMapping` - Used for configuring mapping of `hello` method to incoming HTTP request. In this case `hello` method will map to `/` path with `GET` HTTP method.  




