## Profile specific Property Configurations

To load different properties depending upon environment, Spring allows to create different application.properties file which are profile specific. Convention for such properties file is application-{profile}.properties. By default, only application.properties is loaded and this can be overridden by giving cmd line option `--spring.profiles.active`. 

Code reference: [https://github.com/GlueCoders/springboot-guide/releases/tag/configuration-profiles](https://github.com/GlueCoders/springboot-guide/releases/tag/configuration-profiles)

Following are the contents of `application.properties` and `application-uat.properties` respectively.

```
spring.data.mongodb.uri=mongodb://localhost:27017/test
```

```
spring.data.mongodb.uri=mongodb://localhost:27000/test
```

By default application.properties will be picked. If profile `uat` is passed in cmd line options then application-uat.properties will be picked, cmd is as shown below:

`java -jar <jarname>.jar --spring.profiles.active=uat`


[Prev](/mvc-exceptionmapper.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](#)

