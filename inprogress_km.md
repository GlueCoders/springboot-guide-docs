## About Spring Boot Maven Plugin
      <build>
          <plugins>
              <plugin>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-maven-plugin</artifactId>
              </plugin>
          </plugins>
      </build>  
      
  Spring maven plugin to nest third party jars within package. The spring-boot-starter-parent POM includes <executions> configuration to bind the
repackage goal. If you are not using the parent POM you will need to declare this configuration
yourself as shown below 

 <plugins>
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
            
            Once spring-boot-maven-plugin has been included in your pom.xml it will automatically attempt to rewrite archives to make them executable using the spring-boot:repackage goal. You should configure your project to build a jar or war (as appropriate) using the usual packaging element:
            
## About Spring Parent Pom

Not everyone likes inheriting from the spring-boot-starter-parent POM. You may have your
own corporate standard parent that you need to use, or you may just prefer to explicitly declare all your
Maven configuration.
If you don’t want to use the spring-boot-starter-parent, you can still keep the benefit of the
dependency management (but not the plugin management) by using a scope=import dependency

<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring.version}</version>
            <type>pom</type>
        </dependency>   
        
        Alternative   
        <parent>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-parent</artifactId>
<version>1.5.4.RELEASE</version>
</parent>   

## 

