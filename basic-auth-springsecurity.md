## Spring Security for Authentication and Authorization

Spring Security provides many integrations for implementing security within JEE applications. Here HTTP Basic Authentication will be used along with Role based authorization. In this guide auth is applied on `/books` endpoint. Only authorized members can access the library's catalog of books.  
Code reference: [https://github.com/GlueCoders/springboot-guide/releases/tag/springsecurity-basicauth](https://github.com/GlueCoders/springboot-guide/releases/tag/springsecurity-basicauth)  

### Groundwork 

To make security work, first models are required which will represent member personal information as well as their credentials. Then a registration module is required which will basically allow new users to get onboard the system. Then authorization can be applied on roles for which Spring Security is leveraged.

#### Member and Credentials Model

Member will store the user information like address, contact, name etc. Credentials is for storing username, password and roles that particular user has.  
**Member.java**
```
@Document(collection = "members")
public class Member {

    @Id
    private String id;
    private String firstName;
    private String lastName;
    private Long mobile;
    private String email;
    private LocalDate dob;
    private Address address;
    private List<IssuedBook> issuedBooks;

    public Member() {
    }

```  

**Credentials.java**  
```
@Document(collection = "creds")
public class Credentials {

    @Id
    @JsonIgnore
    private String id;
    @NotNull @NotEmpty
    private String username;
    @Transient @NotNull @NotEmpty
    private String pwd;
    @JsonIgnore
    private String role;
    @JsonIgnore
    private String saltedPwd;
```

Note that role and saltedPwd are ignored from Json serialization while pwd is ignored from database serialization. `saltedPwd` is the salted version of `pwd`, creation of which will be covered in later sections.  

#### Registration Module

For registration, a new REST controller is created `Registration` which will expose a single service `register` and will take `Credentials` as request json. Following is a excerpt from Registration.java file.    

```
@RestController
@RequestMapping(consumes = MediaType.APPLICATION_JSON_UTF8_VALUE,produces = MediaType.APPLICATION_JSON_UTF8_VALUE, path="/unsecured")
public class Registration {

    @Autowired
    private RegistrationService registrationService;
    
    @PostMapping(path = "/register")
    public ResponseEntity<Member> register(@RequestBody Credentials credentials) throws ResourceAlreadyExistsException, ValidationException {
        LOGGER.info("Received registration request for {}",credentials);
        validator.validate(credentials);
        Member member = registrationService.register(credentials);
        LOGGER.info("Member {} registered ",credentials);
        return ResponseEntity.ok(member);
    }

}
```  
`RegistrationService` is the service which will register the credentials received as a `user` in MongoDB. If the username is already present in database, then a conflict (409) will be thrown.  
```

    @Override
    public Member register(Credentials credentials) throws ResourceAlreadyExistsException {
        LOGGER.info("Registering {} in library system", credentials);
        Credentials existingRecord = credentialsDao.findDistinctByUsername(credentials.getUsername());
        if(existingRecord == null) {
            credentials.setSaltedPwd(passwordEncoder.encode(credentials.getPwd()));
            credentials.setRole("USER");
            credentialsDao.save(credentials);
            Member member = new Member();
            member.setEmail(credentials.getUsername());
            memberDao.save(member);
            LOGGER.info("Member {} saved", member);
            return member;
        }else{
            throw new ResourceAlreadyExistsException("Member with given username "+credentials.getUsername() + " already exists");
        }
    }
```  

### Spring Security Configuration

To configure Spring Security first its dependency needs to be included in pom.xml file as shown below.  
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
   <version>${spring.version}</version>
</dependency>
<dependency>
   <groupId>org.springframework.security</groupId>
   <artifactId>spring-security-test</artifactId>
   <version>4.2.3.RELEASE</version>
   <scope>test</scope>
</dependency>
```  

This will include Spring security dependencies related to web and core modules. To configure security `WebSecurityConfigurerAdapter` has to be extended. Only two main items have to be configured one is http authorization rules and other is how to map a given username to existing member( mainly credentials). 

```
@EnableWebSecurity(debug = true)
public class SpringSecurityAdapter extends WebSecurityConfigurerAdapter {
```
Let's explore step by step.

#### Authorization rules
`WebSecurityConfigurerAdapter` has a `configure` method which takes `HttpSecurity` as its parameter. On this parameter authorization rules can be set as shown below. `mvcMatchers` are where patterns of url and methods are written and they are used in the same way as Spring MVC resolves the request urls. Note that in below configuration `httpBasic` authentication is enabled and `csrf` is disabled.

```
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .mvcMatchers(HttpMethod.POST, "/unsecured/register").permitAll()
                .mvcMatchers(HttpMethod.GET, "/books").hasRole("USER")
                .mvcMatchers(HttpMethod.GET, "/books/*").hasRole("USER")
                .and()
                .httpBasic().and()
                .exceptionHandling()
                .authenticationEntryPoint((request, response, e) -> response.sendError(HttpServletResponse.SC_UNAUTHORIZED, e.getMessage()))
                .and()
                .csrf().disable()
                ;
    }
```

#### Custom Authentication Entry Point

In above code snippet there is a `authenticationEntryPoint` method which is basically taking a lambda expression. Since this application is a REST webservice application, on auth failure it makes sense to send a 401 HTTP status code back. By default Spring Security will send back a challenge which will prompt user to enter credentials but in REST this does not make sense. Hence with `authenticationEntryPoint` we override all auth failures to just send an HTTP 401 response.  

#### Custom UserDetailsService

To authenticate a request, Spring Security needs to match username and password present in request with already present credentials in system. For this Spring Security has an interface `UserDetailsService` which has only one method to find user by given username. If user is found and its username and password matches to that of request then request is authenticated else auth error is raised. Below is how it is configued:  
```
	@Autowired
    private PrincipalService principalService;

    @Override
    @Bean
    protected UserDetailsService userDetailsService() {
        return username -> Optional.ofNullable(principalService.findUser(username))
                .map(credential -> User.withUsername(credential.getUsername())
                        .password(credential.getSaltedPwd())
                        .roles(credential.getRole())
                        .build())
                .orElseThrow(() -> new UsernameNotFoundException(username + " not found"));
    }
```  

Here `PrincipalService` is a custom class which just exposes one method `findUser`. `UserDetailsService` is not directly implemented to avoid coupling with Spring Security framework too much.  

#### Password Encoder

Since passwords are going to be stored in database, its mandatory to salt the passwords. For this `BCryptPasswordEncoder` is used which is again provided by Spring Security framework and only its bean has to be returned back and rest is taken care of by framework itself. Ofcourse during saving the password in database, encoding of password has to be done manually by using the password encoder.  
```
@Bean
public PasswordEncoder passwordEncoder(){
  return new BCryptPasswordEncoder();
}
```  

### Testing Spring Security

Now that authentication has been added to REST resources, the existing test cases will fail because existing setup does not send any authentication parameters. To rectify this Spring Security test provides easy annotations with which one can simulate different users in requests.

#### WithMockUser and WithAnonymouseUser

`@WithAnonymousUser` means that no authentication parameters are set in SecurityContext and mvc requests behaves like an unauthenticated request. If according to security config the request is part of authorization then a proper 4xx HTTP code is received back.

`@WithMockUser` means that security context will hold the mock user in authentication parameters. This will act as if request has been authorized by the framework. One can also configure the username, password and role of user by passing the parameters to WithMockUser annotation.

```
@Test
@WithMockUser
public void getAllBooks_NoBooks() throws Exception {
  Behavior.set(bookService).hasNoBooks();
  mvc
         .perform(get("/books"))
         .andExpect(status().isOk())
         .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
         .andExpect(content().json("[]"));
  verify(bookService, times(1)).getAllBooks(any(), any(), any(), any());
}
```

#### Custom Configuration to import Spring Security Configuration

Since Spring Security needs an implementation of `UserDetailsService` which in this case is nothing but a DAO, setting up of unit tests can become troublesome especially when only MvcTest has to be executed. For this a custom test configuration class is created which creates a bean of Dao/Service classes required which are mocked instances of themselves. Then this configuration class is imported in all MvcTest classes which bootstraps Spring Mvc properly.

```
@Configuration
@Import({Oval.class, Validator.class, SpringSecurityAdapter.class})
public class MvcBootstrap {

    @Bean
    public PrincipalService principalService(){
        return Mockito.mock(PrincipalService.class);
    }

    @Bean
    public CredentialsDao credentialsDao(){
        return Mockito.mock(CredentialsDao.class);
    }
}
```

```
@RunWith(SpringRunner.class)
@WebMvcTest(Books.class)
@Import({MvcBootstrap.class})
@WithAnonymousUser
public class BooksTest {

    @Autowired
    private MockMvc mvc;

    @MockBean
    private BookService bookService;
```  



[Prev](/configuration-profiles.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](/basic-auth-springsecurity.md)
