## Getting started with REST using Spring MVC

Here basic REST APIs that are needed for [Library sytem](/domain.md) will be written i.e. CRUD(Create, Retrieve, Update and Delete) REST service on entity `Book`. As per the [specs](/domain.md) the `Retrieve/ Get` method will be available to both Member and Admin, only the representation will differ. For the simplcity of this topic, same representation will be returned in `GET` call. Different representations will be added once security/roles are added to application since the implementation or calls may differ. Also handling for exception scenarios will be added in later topics.  

Code reference: [https://github.com/GlueCoders/springboot-guide/releases/tag/rest-with-mvc](https://github.com/GlueCoders/springboot-guide/releases/tag/rest-with-mvc)  

#### Entity Definition : Book

Below is the definition of Book, this is the representation that will be mapped to REST responses and requests.
```
public class Book implements Serializable {

    private String title;
    private String author;
    private int publishedYear;
    private List<String> categories;
    private String publisher;
    private long isbnCode;
}
```  

#### REST Layer : Books
This class serves as REST layer which will be mapped to HTTP requests. 

```
package org.gluecoders.library.rest;

import org.gluecoders.library.models.Book;
import org.gluecoders.library.services.BookService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController()
@RequestMapping(produces = MediaType.APPLICATION_JSON_UTF8_VALUE, path = "/books")
public class Books {

    @Autowired
    private BookService bookService;

    @GetMapping
    public ResponseEntity<List<Book>> getAllBooks() {
        List<Book> list = bookService.getAllBooks();
        return new ResponseEntity<>(list, HttpStatus.OK);
    }

    @GetMapping("/{isbn}")
    public ResponseEntity<Book> getBook(@PathVariable("isbn") long isbnCode) {
        Book book = bookService.getBookByISBN(isbnCode);
        if(book == null) {
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        }else{
            return new ResponseEntity<>(book, HttpStatus.OK);
        }
    }

    @PostMapping
    public ResponseEntity<Book> addBook(@RequestBody Book book) {
        book = bookService.addBook(book);
        return new ResponseEntity<>(book, HttpStatus.CREATED);
    }

    @DeleteMapping("/{isbn}")
    public ResponseEntity deleteBook(@PathVariable("isbn") long isbnCode) {
        bookService.deleteBook(isbnCode);
        return new ResponseEntity(HttpStatus.OK);
    }

}
```
Let's go to individual lines :
- `@RestController` - Tells Spring to map the methods of this class to HTTP requests.  
- `@RequestMapping` - Defines the HTTP path that should be mapped to the controller class.
- `@GetMapping . @PostMapping, @DeleteMapping` - These annotations are flavoured `@RequestMapping` which defined HTTP methods as well. These annotations have parameters for HTTP path, Content-Type and Accept parameters.  
- `ResponseEntity` - This is used to return response body, headers and status code.

#### Business/Service Layer : BookService
Below is the interface to be consumed by REST layer in our application.

```
public interface BookService {

    List<Book> getAllBooks();

    Book addBook(Book book);

    Book getBookByISBN(long isbn);

    void deleteBook(long isbn);
}
```
Below is the default implementation of this interface, which as of now will do nothing but just return null or empty lists as required from signature.
```
package org.gluecoders.library.services;

import org.gluecoders.library.models.Book;
import org.springframework.stereotype.Component;

import java.util.Collections;
import java.util.List;

@Component
public class DefaultBookService implements BookService {


    @Override
    public List<Book> getAllBooks() {
        return Collections.emptyList();
    }

    @Override
    public Book addBook(Book book) {
        return book;
    }

    @Override
    public Book getBookByISBN(long isbn) {
        return null;
    }

    @Override
    public void deleteBook(long isbn) {

    }
}
```
`@Component` is a Spring annotation which creates a instance of the class and registers it in Bean Registry. Anywhere this bean's instance can be acquired by using @Autowired annotation as shown in REST layer.

[Prev](/quick-hello-world.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](/testing-rest-webmvctest.md)