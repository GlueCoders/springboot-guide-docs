## ActiveMQ Integration  

In this post Apache ActiveMQ will be integrated using Spring JMS libraries. Note that an inmemory broker is used for easy demo purposes.  

### POM Dependency  
Like all other modules Spring Boot has starter for ActiveMQ too. Following are the dependencies required.  
```
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>      
<dependency>
     <groupId>org.apache.activemq</groupId>
     <artifactId>activemq-amqp</artifactId>
     <version>5.15.2</version>
</dependency>
```
`activemq-amqp` is required to use amqp protocol to connect to ActiveMQ otherwise a TCP connection string is expected by the library.

### Property file configuration

Since here inmemory ActiveMQ is used there is no need to specify user, password and brokerUrl.  
```
spring.activemq.in-memory=true
spring.activemq.packages.trustAll=true
```
Second property i.e. `packages.trustAll` is to instruct JMS to trust all packages when it has to do object serialization, otherwise `ClassNotFoundException` will occurr.  

If external broker is being used, one can configure its property as shown below:  
```
spring.activemq.broker-url=amqp://192.168.1.210:9876
spring.activemq.user=admin
spring.activemq.password=secret
```

Check out [ActiveMQProperties](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/activemq/ActiveMQProperties.java) for more options on how to configure ActiveMQ.

### Producer  

Spring JMS ships with [JmsTemplate](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/core/JmsTemplate.html) class which can be used to publish messages to broker.  Boot will autoconfigure JmsTemplate instance and it can be autowired directly in classes which handle publishing work.  
```
import org.springframework.jms.core.JmsTemplate;

@Component
public class JmsSender implements EventHandler{

    private static final Logger LOGGER = LoggerFactory.getLogger(JmsSender.class);

    @Autowired
    private JmsTemplate jmsTemplate;

    @Override
    public void handle(RegistrationEvent event) {
        jmsTemplate.convertAndSend("members", event);
        LOGGER.info("Registration event send to destination : members");
    }
}

```  

Here `EventHanlder` is independent of Jms, it is used to enforce Visitor patter so that each application event has handler associated to it. Below is how `RegistrationEvent` looks like:  
```
public class RegistrationEvent extends Event {

    private final String username;

    public RegistrationEvent(Member member) {
        super(Events.NEW_MEMBER.get());
        username = member.getEmail();
    }

    @Override
    public void accept(EventHandler handler) {
        handler.handle(this);
    }
}
```
`Event` class extended by `RegistrationEvent` implements `Serializable` interface so that Jms API is able to write it out on wire.  

### Consumer  

To create a consumer method has to be designated as `JmsListener` and destination has to be provided. If object serialization is being used then that object can be used as method parameter. However in real production systems, messaging format should be data models agnostic and something like JSON, XML or Apache Thrift should be used. For simplicity here direct data model is being used.  
```
import org.springframework.jms.annotation.JmsListener;

@Component
public class RegistrationEventListener {

    private static final Logger LOGGER = LoggerFactory.getLogger(RegistrationEventListener.class);

    @JmsListener(destination = "members")
    public void receive(RegistrationEvent event){
        LOGGER.info("Received event {} from members queue", event.getEventType());
    }
}
```  

[Prev](/basic-auth-springsecurity.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](/activemq-integration.md)