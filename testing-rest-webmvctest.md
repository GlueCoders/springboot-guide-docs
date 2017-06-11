## Testing REST Controllers with Spring Test

In [previous guide](/rest-with-mvc.md), REST Controller for `Books` is created. It is a good practice to keep the test cases upto speed with development. Here let's explore how to unit tests REST methods.

#### @WebMvcTest  
To test Spring MVC Controllers `@WebMvcTest` annotation is used. This annotation scans for all MVC related components and will not include regular `@Component` classes. This is often used to test one controller class  at a time and is combined with Mockito framework to mock the dependencies. Spring has `@MockBean` annotation which plays nice with Mockito library.

#### Test Class Definition
```
@RunWith(SpringRunner.class)
@WebMvcTest(Books.class)
public class BooksTest {

   @Autowired
   private MockMvc mvc;
   
   @MockBean
   private BookService bookService;
```  
`@RunWith` defines the runner class that is to be used to run test cases, `SpringRunner` is the defacto choice since `Spring` is used pretty much for everything in the application.  
`@WebMvcTest` takes the class of controller that is under test. This will start the web application context, embedded servlet container is not used and this keeps testing lightweight.  
`MockMvc` is the helper class which provides syntax to call controller classes as HTTP requests. It also define methods for expectations on HTTP response.  
`@MockBean` creates a Mockito mock of the `BookService` which is a dependency for `Books` controller. By having mock we can control the behavior of dependencies without calling the real method and really just focus on unit testing of controller class. To control the behavior of `bookService`, a `Behavior` nested class has been defined in this test cases which will be shown later.

#### MockMvc Helper 

##### GET Method

Following test case corresponds to `getAllBooks()` method in `Books` controller.
```
@Test
    public void getAllBooks_NoBooks() throws Exception {
        Behavior.set(bookService).hasNoBooks();
        mvc
                .perform(get("/books"))
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
                .andExpect(content().json("[]"));
        verify(bookService, times(1)).getAllBooks();
    }
```  
`.perform(..` is analogous to making an HTTP request to REST controller. 
`.get("/books")` defines that a HTTP GET request has to be made on '/books' path.
`.andExpect(..` can be used to set expectations on HTTP response received from controller class such as `status().isOk()` means that HTTP response code should be `200`. `content().json([])` means that response body should match json content that is given in `json()` method.
`Behavior` is a custom class which is setting Mockito methods on `bookService`.

##### POST Method

Following test case corresponds to `addBook()` method in `Books` controller.
```
@Test
    public void addBook_Positive() throws Exception {
        Behavior.set(bookService).returnSame();
        String bookContent = mapper.writeValueAsString(effectiveJavaBook);
        mvc
                .perform(post("/books")
                        .content(bookContent)
                        .contentType(MediaType.APPLICATION_JSON_UTF8))
                .andExpect(status().isCreated())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
                .andExpect(content().json(bookContent));
        verify(bookService, times(1)).addBook(effectiveJavaBook);
    }
```  
`.post(..` defines to make an HTTP POST request along with content and content-type defined in methods.
