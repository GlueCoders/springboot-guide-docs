## Deployment

By default Spring Boot packages application as a `jar` and runs an `embedded tomcat`. Let's explore how to use `embedded Jetty` or create a traditional `war` to deploy on external containers.  

### Embedded Jetty

Code reference: [https://github.com/GlueCoders/springboot-guide/releases/tag/embedded-jetty](https://github.com/GlueCoders/springboot-guide/releases/tag/embedded-jetty)  
To use embedded jetty first tomcat dependency has to be excluded in `pom`.  
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <version>${spring.version}</version>
   <exclusions>
      <exclusion>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-tomcat</artifactId>
      </exclusion>
   </exclusions>
</dependency>
```  

Add dependency on `jetty starter`.  
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-jetty</artifactId>
   <version>${spring.version}</version>
</dependency>
```   

It can be verified from the logs that `jetty` is being used as servlet container.  
```
18:22:39.312 [main] INFO  org.eclipse.jetty.server.AbstractConnector - Started ServerConnector@71b6d77f{HTTP/1.1,[http/1.1]}{0.0.0.0:8080}  
18:22:39.312 [main] INFO  org.springframework.boot.context.embedded.jetty.JettyEmbeddedServletContainer - Jetty started on port(s) 8080 (http/1.1)  
18:22:39.347 [main] INFO  org.gluecoders.library.Application - Started Application in 18.59 seconds (JVM running for 22.554)
```  

### Packaging Application as War for external container  

1. Explicitly add scope to tomcat library as provided.

```
 <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-tomcat</artifactId>
   <version>${spring.version}</version>
   <scope>provided</scope>
</dependency>
```

2. Change packaging type to `war`.

```
<packaging>war</packaging>
```

3. Extend `SpringBootServletInitializer` class from `Application`.

```
package org.gluecoders.library;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.support.SpringBootServletInitializer;

@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```  

4. Add web.xml in under src/main/webapp/WEB-INF folder


[Prev](/requestvalidation-oval.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](#)