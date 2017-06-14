# Spring Boot Guide

Step by step introduction to Spring Boot and integration with other Spring projects that are out there. Domain for this project would be an Online Library Portal, below is the basic specs on how application should work. Please refer to [Table of Contents](/TOC.md) for further reading. Refer to [some caveats](/gotchas.md) that one needs to be careful about while using Spring Boot. As of writing this, the Spring Boot version is *1.5.4.RELEASE*.

## Reading Guidelines  
Every topic contains a link to the code which can be downloaded as zip or from Github. The project is Maven based project no extra step is required to setup the project unless explcitly mentioned in some topic. The guide is written in an incremental fashion, though you can skip through the sections if needed.

## Library - Specs for Application

#### Actors  
- Member 
- Admin

#### Actions
- Member  
	1. Has to be registered to be qualified as Member.
    2. Must log in to the library portal to browse the books.
    3. Can search for books by various criterias explained below.
    4. Can checkout/ issue a book if available.
    5. Can return a book.
    6. Can add a notification hook to an already issued book to get notified when that book becomes available.
    7. Log out from the library portal.
    8. Deactivate own account.  
 
- Admin
	1. Cannot register, crendentials have to be provisioned from backend.
    2. Must log in to the library portal for any action.
    3. Can search for books by various criterias explained below, can also see issue status of books.
    4. Can add new books to the catalog. 
    5. Can delete the existing non-issued books.
    6. Notify a Member about the expiry of term of issuance.   

#### Search Criteria for Books  
- By Author
- By Title
- By Categories
- By ISBN Code
- By Year of Publishing

## Feedback
Please feel free to log an issue for any sort of improvement or content required in this guide.  

*authored by Anand Rajneesh*  
[Reference project](https://github.com/GlueCoders/springboot-guide)&nbsp;&nbsp;&nbsp;&nbsp;[Documentation project](https://github.com/GlueCoders/springboot-guide-docs)
