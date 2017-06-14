## MongoDB in Spring Boot using Spring Data  

The goal of Spring Data repository abstraction is to significantly reduce the amount of boilerplate code required to implement data access layers for various persistence stores. MongoDB is a document based storage, which stores data in BSON format which is similar to JSON(WIP).  
Code reference: [https://github.com/GlueCoders/springboot-guide/releases/tag/mongo-springdata](https://github.com/GlueCoders/springboot-guide/releases/tag/mongo-springdata)

#### Spring Data MongoDB Starter POM
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-mongodb</artifactId>
  <version>${spring.version}</version>
</dependency>
```
After adding this dependency Spring Boot will automatically configure defaults for connecting to MongoDB. By default it will try to connect localhost MongoDB at port 27017. To change this configuration, one can modify `spring.data.mongodb.uri` in `application.properties`.  

#### Entity Models - Book

```
import com.fasterxml.jackson.annotation.JsonIgnore;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import java.io.Serializable;
import java.time.YearMonth;

@Document(collection = "books")
public class Book implements Serializable {

    @Id @JsonIgnore
    private String id;
    private String title;
    private String author;
    private YearMonth publishedYear;
    private List<String> categories;
    private String publisher;
    private long isbnCode;
```  
`@Document` defines the collection name in MongoDB in which `Book` documents will be saved.  
`@Id` maps the `_id` field to `id` in `Book`. In this application, `id` is generated automatically by MongoDB.  
`@JsonIgnore` is the Jackson annotation, this is to avoid serialization of id field in json since `isbnCode` is the natural id with which clients will work.  

In MongoDB this object's representation will be as shown below  
```
{
        "_id" : ObjectId("59414737007f68148802a13a"),
        "_class" : "org.gluecoders.library.models.Book",
        "title" : "Effective Java",
        "author" : "Joshua Bloch",
        "publishedYear" : {
                "year" : 2011,
                "month" : 8
        },
        "categories" : [
                "Programming",
                "Java"
        ],
        "publisher" : "noidea",
        "isbnCode" : NumberLong(1234567890)
}
```  

#### Index on MongoDB

Since `isbnCode` is the identifier for Books, let's create an unique index on it to prevent duplicates from getting added.

```
> db.books.createIndex({"isbnCode":1},{"unique":true})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
```

#### Dao inherited from Spring Data

To use Spring Data daos have to extend Repository interface. It could be CrudRepository or PagingRepository or as in this case MongoRepository. The MongoRepository provides many basic implementations by default. To write custom find queries, Spring Data provides two ways one by writing descriptive method names or by defining custom queries in annotation above the methods. More details can be found at [Spring Data reference site](http://docs.spring.io/spring-data/mongodb/docs/current/reference/html/#repositories.query-methods.details)  

```
package org.gluecoders.library.dao;

import org.gluecoders.library.models.Book;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface BookDao extends MongoRepository<Book, String> {
    Book findDistinctByIsbnCode(long isbnCode);
}
```
In given BookDao, `findDistinctByIsbnCode` defines that query with `isbnCode` and find distinct matching document. Since unique index is applied on `isbnCode` the query will always return one Book if present. `@Repository` defines that this interface has to be used as Data Access Layer and Spring will automatically take care of Exception mapping and datasource related dependencies.  


#### Using Dao in Business Layer

Since `BookDao` is annotated with `@Repository`, Spring will create an implementation for it which then can be autowired in business layer as shown below.  
```
@Component
public class DefaultBookService implements BookService {

    private static final Logger LOGGER = LoggerFactory.getLogger(DefaultBookService.class);

    @Autowired
    private BookDao bookDao;

    @Override
    public List<Book> getAllBooks() {
        LOGGER.info("getAllBooks() invoked");
        return bookDao.findAll();
    }
```  

#### Gotchas related to Java 8 DateTime API

Spring Data provides automatic conversion for some Java 8 `DateTime` APIs. In this application,`YearMonth` is used for storing `publishedYear`, for which there is no converter available. To make this work a custom converter has to be written as shown below.  
```
package org.gluecoders.library.config;

import com.mongodb.BasicDBObject;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.convert.converter.Converter;
import org.springframework.data.mongodb.core.convert.CustomConversions;
import java.time.YearMonth;
import java.util.Arrays;

@Configuration
public class Database{

    public enum YearMonthConverter implements Converter<BasicDBObject, YearMonth> {
        INSTANCE;

        private YearMonthConverter(){}

        @Override
        public YearMonth convert(BasicDBObject source) {
            return YearMonth.of(
                    (int) source.get("year"),
                    (int) source.get("month")
            );
        }
    }

    @Bean
    public CustomConversions customConversions(){
        return new CustomConversions(Arrays.asList(YearMonthConverter.INSTANCE));
    }

}
```  

If this converter is not provided then MongoDB will complain about conversion of `BasicDBObject` to `YearMonth` object.  

[Prev](/swagger-docs.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](/testing-mongodb-springdata.md)