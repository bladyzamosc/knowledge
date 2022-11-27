### 1. Injection 

- field
- constructor - rather for mandatory, if there is more than one constructor @Autowired must be used
- method - @Autowired is required
- setter - not mandatory, but @Required makes required setting of the bean
- BeanCurrentlyInCreationException is circular dependency occurs
- in constructor @Lazy tels spring to use proxy and bean is fully created if it will be used

### 2. Autowire

- by type
- by name
- @Primary - bigger priority
- @Qualified - over @Primary
- @Autowired Filter byNameFileter - ByNameFileter implementation will be picked up

### 3. Bean annotations 

- @Bean
- @Component
- @Controller and @RestController - spring servlet knows where to look for mappings
- @Service
- @Repository - hibernate exceptions are translated into DataAccessErrors

### 4. Repository

- injection persistence context

  @PersistenceContext
  EntityManager entityManager;

- spring.jpa.show-sql=true

### 5. Spring data

- Repository -> CrudRepository -> PagingAndSortingRepository -> JpaRepository
- public interface ExampleRepo extends JpaRepository<Example,Integer> {}
- save, fidById, findAll, deleteById

### 6. Design patterns in String

1. Factory pattern - BeanFactory, ApplicationContext
2. Singleton - default bean scope ;)
3. Prototype - prototype-scope beans
4. Proxy pattern - AOP support
5. Template method - RestTemplate, JdbcTemplate, JmsTemplate
6. Data Access Object Pattern - Dao suport
7. MVC - Spring MVC
8. Front controller pattern - DispatcherServlet 
9. View Helper -custom JSP tags
10. Adapter pattern - JMS adapters and JDBC adapters in String integration

### 7. Spring events 

- ContextStartedEvent
- ContextRefreshedEvent - triggered when the context is either initialized or refreshed using the refresh() method
- ContextStoppedEvent - triggered when the context is stopped using the stop() method.
- ContextClosedEvent - triggered when the close() method is called on the ApplicationContext
- RequestHandledEvent - triggered to let all beans know that an HTTP request has been handled

### 8. ApplicationContext subtypes  

ClassPathApplicationXmlContext, AnnotationConfigWebApplicationContext, WebXmlApplicationContext

### 9. ApplicationContext vs BeanFactory

- A - eager initialization
- A - is subtype of BeanFactory
- A supports annotation based dependency injection

### 10. Bean id 

- @Component public class MyBean {} - mybean
- @Component("bean1) public class MyBean {} - bean1

### 11. @Bean vs @Component

- C - autodetect, B - explicit bean creation
- C - class level, B - method level

### 12. Scopes

- Singleton (default always), Prototype (@Scope("prototype"))
- Session, Request, Application, WebSocket - only for web

### 13. Inner bean

- is bean is used as a property of another