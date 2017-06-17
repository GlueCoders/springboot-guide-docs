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

### Packaging Application as War for external container (Serlvet 3.0) 

Code reference: [https://github.com/GlueCoders/springboot-guide/releases/tag/war-packaging](https://github.com/GlueCoders/springboot-guide/releases/tag/war-packaging)  

- Explicitly add scope to tomcat library as provided.

```
 <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-tomcat</artifactId>
   <version>${spring.version}</version>
   <scope>provided</scope>
</dependency>
```

- Change packaging type to `war`.

```
<packaging>war</packaging>
```

- Extend `SpringBootServletInitializer` class from `Application`.

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

- Add web.xml in under src/main/webapp/WEB-INF folder.

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

    <servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
        </init-param>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>org.gluecoders.library.config</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>appServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    <welcome-file-list>
        <welcome-file></welcome-file>
    </welcome-file-list>

</web-app>
```

Run mvn package and a war will be created in target folder.  



[Prev](/requestvalidation-oval.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](/custom-mongodb.md)
