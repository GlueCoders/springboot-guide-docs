## Unit Testing Mongo Repositories

To test Spring Data for MongoDB, let's use Fongo (an in-memory implementation of MongoDB meant for mocking) and Nosqlunit for MongoDB for setting up for fixtures.  
Code reference: [https://github.com/GlueCoders/springboot-guide/releases/tag/fongo-testingmongo](https://github.com/GlueCoders/springboot-guide/releases/tag/fongo-testingmongo)    

### Fongo (Fake Mongo) Setup

Add pom dependency as given below.  
```
<dependency>
    <groupId>com.github.fakemongo</groupId>
    <artifactId>fongo</artifactId>
    <version>2.0.6</version>
    <scope>test</scope>
</dependency>
```  

A custom configuration class is required in test package to wire Fongo as mock MongoDB as shown below.  
```
package org.gluecoders.library.dao.config;

import com.github.fakemongo.Fongo;
import com.mongodb.MongoClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.config.AbstractMongoConfiguration;

@Configuration
public class FakeMongo extends AbstractMongoConfiguration{

    @Override
    protected String getDatabaseName() {
        return "mockDB";
    }

    @Bean
    public MongoClient mongo() {
        Fongo fongo = new Fongo("mockDB");
        return fongo.getMongo();
    }

}
```  
This class extends `AbstractMongoConfiguration` so that during Junit run, `MongoTemplate` and other Mongo related dependecies are taken care of automatically. In `mongo()` method `Fongo`'s instance is returned which will act as mock MongoDB.

### Nosqlunit-MongoDB Setup
```
<dependency>
    <groupId>com.lordofthejars</groupId>
    <artifactId>nosqlunit-mongodb</artifactId>
    <version>0.7.6</version>
    <scope>test</scope>
</dependency>
```  

For nosqlunit-mongodb Junit Rule has to be wired in Test Class.
```
@Rule
public MongoDbRule embeddedMongoDbRule = newMongoDbRule().defaultSpringMongoDb("mockDB");
```  

### Setup of DAO in test class

```
@RunWith(SpringRunner.class)
@Import(value = {FakeMongo.class, Database.class})
@EnableMongoRepositories(basePackageClasses = {BookDao.class})
public class BookDaoTest {
```
`@Import`  imports the configuration from the given classes, in this case `FakeMongo` provides `Fongo` instance and `Database` provides the custom converter for `YearMonth` class.  
`@EnableMongoRepositories` tells Spring to look for Mongo dependencies and enable `BookDao` to act as DAO layer with MongoDB as storage.

### Usage of nosqlunit for fixtures

There are two basic annotations which can be used to set fixtures and assert database state after a test has run.  
`@UsingDataSet(locations, loadStrategy)` - ensures the database has data loaded from file given in `locations` parameter. `LoadStrategy` are of three types : `CLEAN_INSERT`(clean and insert), `DELETE_ALL`(delete everything) and `INSERT`(append on existing data).
`@ShouldMatchDataSet(location)` defines that data present in database should match the data given in file(`location` parameter). This check is done after a test case has run.

### Writing unit tests

Let's look at some of the test cases one by one.  
``` 
@Test
@UsingDataSet(loadStrategy = LoadStrategyEnum.DELETE_ALL)
public void getAllBooks_NoBooks() {
   List<Book> books = bookDao.findAll();
   assertTrue("Returned book list should be empty", books.isEmpty());
}
```  
This ensures that database should be clean when the test cases start, hence the test also expects that db should not return any books in the list.  

```
@Test
@UsingDataSet(loadStrategy = LoadStrategyEnum.CLEAN_INSERT, locations = "/books/books.json")
public void getAllBooks_WithBooks() {
   List<Book> books = bookDao.findAll();
   assertFalse("Returned book list should not be empty", books.isEmpty());
}
```  
This test case first loads the data from `/books/books.json` which is present in `/test/resources` folder. After setting up of data, the test case runs and asserts that database should return some list of books.  

```
@Test
@UsingDataSet(loadStrategy = LoadStrategyEnum.CLEAN_INSERT, locations = "/books/empty.json")
@ShouldMatchDataSet(location = "/books/book1.json")
public void addBook(){
   Book book = Book.builder()
         .author("Joshua Bloch")
         .categories("Programming", "Java")
         .isbn(1234567890L)
         .publisher("Addison-Wesley")
         .yearOfPublishing(2008, Month.MAY)
         .title("Effective Java")
         .build();
  bookDao.save(book);
}
``` 

This will first load an empty json in database to ensure everything is clean. Then test case will run which will insert a book. Then `@ShouldMatchDataSet` will match the database data with the `/books/book1.json` and if the data matches the test case will pass. Below is the book1.json file which is used by this test case.  
**/books/book1.json**
```
{
  "books": [
    {
      "title": "Effective Java",
      "author": "Joshua Bloch",
      "publishedYear": {
        "year": 2008,
        "month": 5
      },
      "categories": [
        "Programming",
        "Java"
      ],
      "publisher": "Addison-Wesley",
      "isbnCode": 1234567890
    }
  ]
}
```  

[Prev](/mongodb-basics.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](#)
