https://github.com/onsever/spring-framework-notes  

https://towardsdev.com/30-days-of-spring-boot-day-25-interceptors-and-filters-6daba225f37c  

https://medium.com/@gaddamnaveen192/spring-boot-interview-prep-300-questions-and-real-world-challenges-explained-c1f5ab7bb4b1  
https://medium.com/@sylvain.tiset/top-10-microservices-design-patterns-you-should-know-1bac6a7d6218  

https://www.springcloud.io/post/2023-06/spring-boot-annotation/#gsc.tab=0  


## Spring Security 

https://medium.com/@dilankacm/spring-security-architecture-explained-with-jwt-authentication-example-spring-boot-5cc583a9aeac  
https://medium.com/@tericcabrel/implement-jwt-authentication-in-a-spring-boot-3-application-5839e4fd8fac  


### Distributed Transactions in Microservices  https://archive.is/3OYwF#selection-591.0-591.525

Spring Boot provides several features to manage distributed transactions in a microservices architecture.
 Spring Boot can use either the 2PC or Saga pattern to manage distributed transactions, depending on the specific needs of the application.
One way to implement 2PC in Spring Boot is to use the XA (eXtended Architecture) protocol, which allows multiple databases to participate 
in a single transaction. Spring Boot provides support for XA transactions through the Atomikos and Bitronix transaction managers,
 which can be used to coordinate transactions across multiple services.
Two-phase commit (2PC) is a traditional approach that relies on a centralized coordinator to manage the transaction across multiple services.
It ensures that all services commit or rollback the transaction atomically, ensuring data consistency. However, 2PC has several disadvantages 
in a microservices architecture, including performance, complexity, single point of failure, and locking.
 
The Saga pattern, on the other hand, is a more recent approach that uses a series of local transactions that are coordinated to achieve the overall transaction.
It is designed to be more fault-tolerant and scalable, as there is no centralized coordinator, and each service handles its own local transaction. 
Sagas are also able to handle partial failures and can recover from them without rolling back the entire transaction. However, Sagas can be more complex
to implement and manage, and data consistency is not guaranteed.

For @Configuration/ @AutoConfiguration  classes  need to add in folder src/main/resources/META-INF/spring/ in  file  org.springframework.boot.autoconfigure.AutoConfiguration.imports  with full  qualifiedname without extension
