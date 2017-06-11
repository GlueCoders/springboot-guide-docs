## Logging in Spring Boot

By default Spring Boot includes `logback` and `slf4j` dependencies, so any application can use `SLF4J` libraries for logging. Default configuration is to log output in console.  

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


