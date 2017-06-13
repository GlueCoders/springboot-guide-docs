## Logging in Spring Boot

By default Spring Boot includes `logback` and `slf4j` dependencies, so any application can use `SLF4J` libraries for logging. Default configuration is to log output in console.  
Code reference: [https://github.com/GlueCoders/springboot-guide/releases/tag/logging](https://github.com/GlueCoders/springboot-guide/releases/tag/logging)
#### Output to file  
To output in a file add following line to the `application.properties` in `src/main/resources` folder.    
```
logging.file=custom.log
```  
Log file rotate automatically after 10mb is reached.  

#### Log Levels  
To control log levels, it can be done by adding package controls in `application.properties` as shown below.  
```
logging.level.root=WARN
logging.level.org.springframework.web=INFO
logging.level.org.gluecodders.library=DEBUG
```  

#### Log Patterns  
`logging.pattern.file` for logging pattern for file appenders. Similarly `logging.pattern.console` for logging pattern for console appenders.  

### Log4j2
Let's include log4j2 in the project and look at the available configurations.

#### POM Dependency
```
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter</artifactId>
       <version>${spring.version}</version>
       <exclusions>
            <exclusion>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-logging</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-log4j2</artifactId>
       <version>${spring.version}</version>
   </dependency>
```  

#### Basic Configuration  

Log4j2 supports xml, yaml, json and properties configuration. For verbosity let's explore XML configuration.  

For simple console output below xml will work fine.  
```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <logger name="org.gluecoders.library" level="debug" additivity="false">
            <AppenderRef ref="Console"/>
        </logger>
        <Root level="info">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```  
This configuration defines two loggers one is RootLogger and other is for application's package org.gluecoders.library. Only console appender is declared here, so all the logs will be logged to console output.  

#### JSON Layout Configuration  
```
<Appenders>
   <Console name="Console" target="SYSTEM_OUT">
      <JsonLayout compact="true" eventEol="true"/>
   </Console>
</Appenders>  
```
This will log statements in json format which might be useful if the log output goes to analytics platform like Elasticsearch.  

#### File Appender Configuration
```
<File  name="File" fileName="app.log">
   <JsonLayout compact="true" eventEol="true"/>
</File>
```

#### Rolling File Appendr Configuration
```
<RollingFile name="Rolling" fileName="app.log" filePattern="target/rolling1/app.%i.log.gz">
   <JsonLayout compact="true" eventEol="true"/>
   <SizeBasedTriggeringPolicy size="500" />
</RollingFile>
```

#### Configuring logging for different packages  
```
<logger name="org.springframework" level="error" additivity="false">
   <AppenderRef ref="Console"/>
</logger>
```  
To control logging of third party libraries and other packages, different logger configurations can be added as shown above.  

[Prev](/testing-rest-webmvctest.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](/swagger-docs.md)
