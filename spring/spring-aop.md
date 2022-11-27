### 1. Usage 

- security, logging, transaction, management, auditing, error handling, performance tracking 
- cross-cutting concerns, can be usefull to track something across different layers

### 2. spring-aop

- less powerful than AspectJ
- spring-boot-starter-aop

### 3. Terminology

- Aspect - @Aspect - class implementing cross-cutting concerns 
- Pointcut - @Pointcut - expression determining what method will be intercepted 
- Advice - inside @Aspect class, logic which will be invoked 
  - @Before
  - @After - generic annotation for both positive and negative (exception) scenarios 
  - @AfterReturning - if method successfully finished
  - @AfterThrowing - after throwing the exception
  - @Around - for performance tests for example 
- Joinpoint - all methods calls that are intercepted are joinpoints 
- Weaving - the process of implementing AOP around method calls. 
- Weaver - framework ensures that an aspect is invoked at the right time is called a weaver

### 4. Example

<code>
@Aspect<br>
@Configuration<br>
public class AccessCheckAspect {

	private Logger logger = LoggerFactory.getLogger(this.getClass());
	
	@Before("execution(* io.datajek.springaop.movierecommenderaop.business.*.*(..))")
	public void before(JoinPoint joinPoint) {
		logger.info("Intercepted call before execution of: {}", joinPoint);	
	}}
</code>

### 5. Pointcut expressions

- all methods in the package - @Before("execution(* com.app.example.*.*(..))")
- all methods with return type - @Before("execution(String com.app.example..*.*(..))")
- specific method - @Before("execution(String com.app.example..*.*Filtering(..))")
- specific argument - @Before("execution(String com.app.example..*.*(String,..))")
- combined - @Before("execution(String com.app.example..*.*(String,..)) || execution(String com.app.example..*.*Filtering(..))")

### 6. Around example 

<code>
@Aspect

@Configuration
public class ExecutionTimeAspect {

	private Logger logger = LoggerFactory.getLogger(this.getClass());

	@Around("execution(* io.datajek.springaop.movierecommenderaop..*.*(..))")
	public Object calculateExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
		//note start time
		long startTime = System.currentTimeMillis();
		//allow method call to execute
		Object returnValue = joinPoint.proceed();
		//time taken = end time - start time
		long timeTaken = System.currentTimeMillis() - startTime;
		
		logger.info("Time taken by {} to complete execution is: {}", joinPoint, timeTaken);
		return returnValue;
	}}
</code>

### 7. Examples

- @Pointcut("bean(movie*)")
- @Pointcut("io.datajek.springaop.movierecommenderaop.aspect.JoinPointConfig.dataLayerPointcut() || io.datajek.springaop.movierecommenderaop.aspect.JoinPointConfig.businessLayerPointcut()")


