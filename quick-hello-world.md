## Hello World Example

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
Now run this class from IDE, Spring Boot logs should appear in console as shown below  
![Spring Boot logs]({{site.baseurl}}/Capture.PNG)  

On browser hit [http://localhost:8080/](http://localhost:8080/), and a response will appear in return with text "Hi ! Welcome to Spring Boot Guide". 
