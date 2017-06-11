## Testing REST Controllers with Spring Test

In [previous guide](/rest-with-mvc.md), REST Controller for `Books` was created. It is a good practice to keep the test cases upto speed with development. Here let's explore how to unit test REST methods.  
Code reference: [https://github.com/GlueCoders/springboot-guide/releases/tag/rest-with-mvc](https://github.com/GlueCoders/springboot-guide/releases/tag/rest-with-mvc)

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
`get("/books")` defines that a HTTP GET request has to be made on '/books' path.  
`.andExpect(..` can be used to set expectations on HTTP response received from controller class such as `status().isOk()` means that HTTP response code should be `200`.  
`content().json([])` means that response body should match json content that is given in `json()` method.  
`content().contentType(...` means that `Content-Type` header should match the value as given in method.  
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
`post(..` defines to make an HTTP POST request along with content and content-type defined in methods.  
`.mapper` here is the instance of `Jackson` `ObjectMapper` which allows for serialization and deserialization of POJOs to and from JSON.  

##### DELETE Method

Following test case corresponds to `deleteBook()` method in `Books` controller.
```
@Test
    public void deleteBook() throws Exception {
        mvc
                .perform(delete("/books/1234567899"))
                .andExpect(status().isOk());
        verify(bookService, times(1)).deleteBook(anyLong());
    }
```  
`delete("/books...")` defines to make an HTTP DELETE request.  

#### Mockito Usage  

Let's also explore some of the Mockito methods that are being used in this class.

##### verify()  
`verify` method is used to verify the number of times a mock method has been called. This is especially useful to test conditional statements, since if test case does not expect that  code flow to enter a certain condition statement then the methods inside that block should not be called even once.  
It's signature is `verify(T mock, VerificationMode mode)` where `T` corresponds to mocked bean and `mode` refers to number of times methods should be called. It can also be used to match the parameters received by mock method as shown below  
```
verify(bookService, times(1)).getBookByISBN(effectiveJavaBook.getIsbnCode());
```  
This means that method `getBookByISBN` of bean `bookService` should be called only once during execution of test case. Also the parameter received by the method is verified by giving the exact argument as here `effectiveJavaBook.getIsbnCode()`.  

##### when() doReturn() thenReturn()
These constructs are used to set expectations on mocked beans. Let's see an example first :   
```
when(bookService.getBookByISBN(book.getIsbnCode())).thenReturn(book);
when(bookService.addBook(book)).thenReturn(book);
when(bookService.getAllBooks()).thenReturn(Collections.emptyList());
when(bookService.addBook(any())).thenAnswer(invocationOnMock -> invocationOnMock.getArguments()[0]);
```  
First example defines that if `getBookByISBN` is invoked with some `isbnCode` then it should return the same book, since the `isbnCode` is also fetched from the same instance of `book`.  
Second example defines that `addBook` should return the `book` instance if it is called with the same `book` instance.  
Third example defines that `getAllBooks` should return `emptyList` if it is invoked.  
Fourth example defines that `addBook` should return the same instance that it receives as parameter. Note here there is no predefined `book` instance so this will work for any argument passed to `addBook`, however in second example it will only work for that instance of `book` which has been used in `when(..` clause.  

The whole class `BooksTest` can be referred in `src/tests/java` directory under `org.gluecoders.library.rest` package.  

[Prev](/rest-with-mvc.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](#)


