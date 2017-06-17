## Extending MongoDB repositories by writing custom implementations

Let's implement search functionality for Books. As defined in the [specs](/domain-of-guide.md) one can search for books by categories, title, author and publishedYear.

### Text Index on Author and Title
Since title and author fields will be searched using text operator, MongoDB will require an index created as below:

```
db.createIndex({"author":"text", "title":"text"})
```

### Custom Dao

To provide custom implementation three steps are required:
1. New interface which defines custom methods to be implemented
2. Implementation of the interface mentioned above.
3. Default Repository Dao extends the custom dao.

##### Step 1 : Defining Custom Dao

Let's create an interface `BookDaoCustom` which defines method to `findBooks`.
```
package org.gluecoders.library.dao;

import org.gluecoders.library.models.Book;
import java.util.List;

public interface BookDaoCustom {

    public List<Book> findBooks(List<String> categories, String author, String title, String publishedYear);
}
```  

##### Step2 : Implementation of Custom Dao

Following is the implementation of `BooksCustomDao`. The class is annotated with `@Component` annotation so that Spring can wire its bean wherever `BookCustomDao` is needed. **The name of class is extremely important it should be derived from default dao. Like in this case the default dao is `BookDao` so the class name is `BookDaoImpl`. If this is not followed then custom implementations will not work at all**
```
package org.gluecoders.library.dao.impl;

import com.sun.org.apache.xml.internal.resolver.readers.TextCatalogReader;
import org.gluecoders.library.dao.BookDaoCustom;
import org.gluecoders.library.models.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.*;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import java.util.List;

@Component
public class BookDaoImpl implements BookDaoCustom {

    private static final Logger LOGGER = LoggerFactory.getLogger(BookDaoImpl.class);

    private final MongoTemplate mongoTemplate;

    @Autowired
    public BookDaoImpl(MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
    }

    @Override
    public List<Book> findBooks(List<String> categories, String author, String title, String publishedYear) 	{
        Query query = buildSearchQuery(categories, author, title, publishedYear);
        LOGGER.debug("Query built for findBooks {}", query);
        return mongoTemplate.find(query, Book.class);
    }

    private Query buildSearchQuery(List<String> categories, String author, String title, String publishedYear) {
        TextCriteria textCriteria = TextCriteria.forDefaultLanguage().matchingAny(author, title);
        Criteria criteria = new Criteria();
        if (categories != null && categories.size() > 0) {
            criteria.and("categories").in(categories);
        }
        if (StringUtils.hasText(publishedYear)) {
            criteria.and("publishedYear.year").is(Integer.parseInt(publishedYear));
        }
        if(author != null || title != null){
            return TextQuery.queryText(textCriteria).addCriteria(criteria);
        }else {
            return Query.query(criteria);
        }
    }
}
```

##### Step 3: Extend custom dao in default Repository Dao

To make use of custom dao, Spring Data requires it to be extended by default dao annotated with `@Repository` annotation.

```
@Repository
public interface BookDao extends MongoRepository<Book, String>, BookDaoCustom {
```

Now the same `bookDao` instance can be used to invoke `findBooks()` method and the implementation from `BookDaoImpl` will work.
```
bookDao.findBooks(....
```


[Prev](/deployment-options.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[TOC](/TOC.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Next](#)
