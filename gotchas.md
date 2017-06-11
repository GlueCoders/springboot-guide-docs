## Some of the Gotchas that one needs to be careful about

1. **Default Package** - When a class doesn’t include a package declaration it is considered to be in the “default package”. The use of the “default package” is generally discouraged, and should be avoided. It can cause particular problems for Spring Boot applications that use `@ComponentScan`, `@EntityScan` or `@SpringBootApplication` annotations, since every class from every jar, will be read.  

2. **Unit Testing with WebMvcTest and JsonTest** - A unit test class cannot have both `@WebMcvTest` and `@JsonTest` annotations. It is better to use `ObjectMapper` explicitly for conversion of JSON and POJOs.  

3. 
