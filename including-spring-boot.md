## Including Spring Boot in Project [Maven]

There are two ways to include and manage Spring Boot dependencies in project. One way is to inherit from Spring Boot parent pom, other way is to include a dependency on `spring-boot-dependencies`.   
Code reference : [https://github.com/GlueCoders/springboot-guide/releases/tag/includespringboot](https://github.com/GlueCoders/springboot-guide/releases/tag/includespringboot)  

### Spring Boot Parent Pom  

By inheriting from Spring Boot parent pom, all the dependencies and plugin are managed by the version of parent pom. To inherit from parent pom following has to be added in pom.xml  
```
  <parent>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-parent</artifactId>
  	<version>1.5.4.RELEASE</version>
  </parent>  
```  
### spring-boot-dependencies  
In case one cannot inherit Spring Boot parent pom, then alternative is to declare a dependency on `spring-boot-dependencies` under `dependencies` tag as shown below  
```
  <dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-dependencies</artifactId>
  	<version>${spring.version}</version>
  	<type>pom</type>
  </dependency>  
```  
However this does not manages versions for plugins, in this case version has to be explicitly set for plugins.  

Note : Including Parent pom or `spring-boot-dependencies` does not actually bring in any dependencies, they are just mechanisms to manage dependencies related to Spring Boot.

### Spring Boot Maven Plugin

Spring Boot Maven plugin runs in package phase to include jars in one package, so that application can be bundled as executable jar. Following has to be added in `plugins` for including this plugin.  
```
  <plugin>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-maven-plugin</artifactId>
  </plugin>
  ```
If Spring Boot parent pom is not inherited, then one has to specify execution phase and plugin version explicitly as shown below   
```
<plugin>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-maven-plugin</artifactId>
	<version>${spring.version}</version>
	<executions>
		<execution>
			<goals>
				<goal>repackage</goal>
			</goals>
		</execution>
	</executions>
</plugin>  
```
### End Note  
As of now project has just barebones, in next steps we will add some code so that we can see it in action.
