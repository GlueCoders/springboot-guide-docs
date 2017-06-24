## Exception Mappers in Spring MVC

Purpose of exception mappers is to segregate handling of exceptions and their mapping to REST responses in a single layer. Let's explore how `Exception` classes and `Handlers` can be tied together to create a ExceptionMappers for all `controllers`.  
Code reference: [https://github.com/GlueCoders/springboot-guide/releases/tag/springmvc-filters](https://github.com/GlueCoders/springboot-guide/releases/tag/springmvc-filters)  

### Context - OVal Validation

In [earlier post](/requestvalidation-oval.md), `OVal` was added to `Book` model for validating request parameters and the controller method `addBook` is responsible for mapping validation failure to `Bad Request` as shown below. This works fine, but can be improved by adding `ExceptionMappers` to the design. Since validation is a cross cutting concern the mapping of validation failure to HTTP response should be tackled in some extensible fashion. Moreover there can be more exception scenarios like database errors, runtime errors which have to be handled in a graceful manner. To handle this, let's explore a bit on how to create Exception hierarchy and `@RestControllerAdvice` class to handle the exceptions.

```
@PostMapping
public ResponseEntity<Book> addBook(@RequestBody Book book) {
  List<ConstraintViolation> violations = validator.validate(book);
  if (!violations.isEmpty()) {
    LOGGER.info("violations {} for request payload {}", violations, book);
    return new ResponseEntity<>(HttpStatus.BAD_REQUEST);
  }
  LOGGER.info("addBook invoked for Book {}", book);
  book = bookService.addBook(book);
  return new ResponseEntity<>(book, HttpStatus.CREATED);
}
```

### Exception to REST response mapping

Since exceptions have to be ultimately mapped to some HTTP response code, it makes sense to have `getStatusCode` type of method in all the exception classes that can be expected by REST layer to handle. Here an abstract class `ResourceException` is created which will serve as parent for all those exceptions that should be propagated to the REST layer.   
```
package org.gluecoders.library.exceptions;

import java.util.function.IntSupplier;

public abstract class ResourceException extends Exception{

    private final IntSupplier statusCodeSupplier;

    public ResourceException(String message, IntSupplier statusCodeSupplier) {
        super(message);
        this.statusCodeSupplier = statusCodeSupplier;
    }
    
    public final int statusCode(){
        return statusCodeSupplier.getAsInt();
    }
}
```

`ValidationException` is one such exception which will be raised when validation failure happens and needs to be handled at controller's level.
```
package org.gluecoders.library.exceptions;

import net.sf.oval.ConstraintViolation;

import java.util.List;
import java.util.stream.Collectors;

public class ValidationException extends ResourceException {

    public ValidationException(String message, StatusCode statusCode) {
        super(message, statusCode);
    }

    public static ValidationException of(List<ConstraintViolation> violations){
       return of(violations, StatusCode.BAD_REQUEST);
    }

    public static ValidationException of(List<ConstraintViolation> violations, StatusCode statusCode){
        String message = violations.stream()
                .map(ConstraintViolation::getMessage)
                .collect(Collectors.joining(","));
        return new ValidationException(message, statusCode);
    }
}
```

Note here `StatusCode` is an enum which is implementing `IntSupplier` as shown below and is used to increase the readability and maintainability of code.
```
package org.gluecoders.library.exceptions;

import java.util.function.IntSupplier;

public enum StatusCode implements IntSupplier{
    BAD_REQUEST(400),
    INTERNAL_SERVER_ERROR(500);

    @Override
    public int getAsInt() {
        return this.statusCode;
    }

    private int statusCode;

    StatusCode(int statusCode) {
        this.statusCode = statusCode;
    }
}
```

### ExceptionHandler - RestControllerAdvice

`RestControllerAdvice` annotated classes can define `ExceptionHandler` which will catch the exception(provided in annotation value) from all the controllers by default.  
```
package org.gluecoders.library.rest.helper;

import org.gluecoders.library.exceptions.ResourceException;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class ResourceExceptionMapper {

    @ExceptionHandler(ResourceException.class)
    public ResponseEntity handle(ResourceException e){
        return ResponseEntity
                .status(e.statusCode())
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .body(String.format("{\"error\":\"%s\"}", e.getMessage()));
    }
}
```

Here `ResourceExceptionMapper` is responsible for handling all `ResourceException`. Once exception is received, it is mapped to `ResponseEntity` using `statusCode` and `getMessage` methods.

### Custom Validator Implementation

To control the exceptions that are raised from validation failures, a custom implementation is used as shown below. It's validating the object received in HTTP request with OVal and if `list` of `ConstraintViolation` is not empty then it throws an `exception` which is again supplied as an `ExceptionSupplier`.  

```
package org.gluecoders.library.rest.helper;

import net.sf.oval.ConstraintViolation;
import org.gluecoders.library.exceptions.ResourceException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.function.Function;

@Component
public final class Validator {

    private final net.sf.oval.Validator validator;

    private static final Logger LOGGER = LoggerFactory.getLogger(Validator.class);

    @Autowired
    public Validator(net.sf.oval.Validator validator) {
        this.validator = validator;
    }

    private <T> List<ConstraintViolation> check(T t){
        return validator.validate(t);
    }

    public <T, E extends ResourceException> void validate(T t, Function<List<ConstraintViolation>, E>     exceptionSupplier) throws E {
        List<ConstraintViolation> violations = check(t);
        if(!violations.isEmpty()){
            LOGGER.error("{} has {} validation errors", t, violations.size());
            throw exceptionSupplier.apply(violations);
        }
    }

}
```


### Change in REST Controllers

After addition of `RestControllerAdvice` classes, `RestController` classes don't have to manage the exceptions themselves. Simply throwing the exception from the controller method will delegate the handle to exception handler.  

```
@PostMapping
public ResponseEntity<Book> addBook(@RequestBody Book book) throws ValidationException {
   validator.validate(book, ValidationException::of);
   LOGGER.info("addBook invoked for Book {}", book);
   book = bookService.addBook(book);
   return new ResponseEntity<>(book, HttpStatus.CREATED);
}
```

### Advance Customization of Exception Handlers

By default the methods in an `@RestControllerAdvice` apply globally to all Controllers. Use selectors `annotations()`, `basePackageClasses()`, and `basePackages()` to define a more narrow subset of targeted Controllers. If multiple selectors are declared, OR logic is applied, meaning selected Controllers should match at least one selector.  


[Prev](/custom-mongodb.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](/configuration-profiles.md)