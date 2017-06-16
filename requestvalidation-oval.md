## Validation of Request Models using OVal  

OVal is a validation framework which works by annotating `beans` with constraints.  
Code reference: [https://github.com/GlueCoders/springboot-guide/releases/tag/basicvalidation-oval](https://github.com/GlueCoders/springboot-guide/releases/tag/basicvalidation-oval)  

### POM dependency  
```
<dependency>
   <groupId>net.sf.oval</groupId>
   <artifactId>oval</artifactId>
   <version>1.87</version>
</dependency>
```  

### Annotations in Bean

There are various annotations provided by OVal which sit under `net.sf.oval.constraint` package. Below are standard `@NotNull` and `@NotEmpty` constraints added to `Book` model.  
```
public class Book implements Serializable {

    @Id @JsonIgnore
    private String id;
    
    @NotNull @NotEmpty
    private String title;
    
    @NotNull @NotEmpty
    private String author;
    
    @NotNull
    private YearMonth publishedYear;
    
    private List<String> categories;
    
    @NotNull @NotEmpty
    private String publisher;
    
    @Min(1000000000)
    private long isbnCode;
```  

### Validating Bean
Validating bean is as simple as creating a `net.sf.oval.Validator` object and calling its validate method. This method returns a list of `ConstraintViolation` which encapsulates all the data related to validation failures.
```
Book book = ...
net.sf.oval.Validator validator = new Validator();
List<ConstraintViolation> violations = validator.validate(book);
```

### How to use in Spring Boot  
Since `Validator` is thread safe, let's create a `bean` for it from a configuration class and autowire it in our controllers. So first line of all controllers would be to validate the incoming model.

```
package org.gluecoders.library.config;

import net.sf.oval.Validator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class Oval {

    @Bean
    public Validator validator(){
        return new Validator();
    }

}
```

Below is the controller class using the `Validator` instance.

```
@RestController
@RequestMapping(produces = MediaType.APPLICATION_JSON_UTF8_VALUE, path = "/books")
public class Books {

    private final static Logger LOGGER = LoggerFactory.getLogger(Books.class);

    @Autowired
    private BookService bookService;

    @Autowired
    private Validator validator;
```

```
@PostMapping
public ResponseEntity<Book> addBook(@RequestBody Book book) {
   List<ConstraintViolation> violations = validator.validate(book);
   if(!violations.isEmpty()) {
      LOGGER.info("violations {} for request payload {}", violations, book);
      return new ResponseEntity<>(HttpStatus.BAD_REQUEST);
   }
   LOGGER.info("addBook invoked for Book {}", book);
   book = bookService.addBook(book);
   return new ResponseEntity<>(book, HttpStatus.CREATED);
}
```  
Now if validation fails for received request, then controller will return a `400 Bad Request` response.


Let's also add a test case for `BadRequest` in `BooksTest` that was created in [REST introduction](/testing-rest-webmvctest.md).

```
@Test
public void addBook_MissingParams() throws Exception {
   Behavior.set(bookService).returnSame();
   String bookContent = mapper.writeValueAsString(incompleteBook);
   mvc
        .perform(post("/books")
            .content(bookContent)
            .contentType(MediaType.APPLICATION_JSON_UTF8))
        .andExpect(status().isBadRequest());
   verify(bookService, never()).addBook(incompleteBook);
}
```
Here an incompleteBook(some parameters are null) is send in request, and it is expected that `400` response will be returned.

[Prev](/testing-mongodb-springdata.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](#)

