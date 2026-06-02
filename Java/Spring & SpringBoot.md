## Starting with SpringBoot
#### Embedded Servers
- SpringBoot provides you with embedded servers(Tomcat, Jetty, Undertow). No need to install servers separately.
### Opening Applications
- Download and extract the files.
### Creating a RestController
- You create your application with a start file.(Name of your app).
- In the same directory create another folder/package. Inside it create any class.
```Java
@RestController  
public class NewRestController {  
    @GetMapping("/")  
    public String sayHello(){  
        return "Hello World!!!";  
    }  
}
```
### Spring Projects
- Spring module build on top of spring framework. -> Like node/npm modules
- Spring Cloud, Spring Data, Spring Batch, Spring Security, Spring LDAP, Spring Services etc.
- Go to www.spring.io 
### Maven
- It is a project management tool -> Manages builds and dependencies.
- Before building the project, you define dependencies in - *pom.xml*
- **Maven Local Repo** - stores downloaded dependencies. Reuses these repos for multiple projects.
- **Maven Central Repo** - Stores all projects and your local repo downloads dependencies from here.
#### pom.xml
- **Project Object Model** file -> Contains list of dependencies.
- Has 3 sections - Project Metadata, dependencies and plugins.
- Project Metadata has Project Coordinates - 
	- *groupId* - Owner of the project.(Usually the package)
	- *artifactId* - Name of project
	- *version*
##### adding dependencies
- add \<dependency> with project coords in the dependency section of pom.xml file. 
- Project coords can be gotten from https://central.sonatype.com/
#### mvnw.sh, mvnw.cmd files
- Used to run projects where Maven is absent. Run mvnw.sh to install correct maven version and run the proj.
```
mvnw clean compile test
```
#### Directory Structure
```
project/
 ├── src/
 │    ├── main/
 │    │    ├── java/         → Application source code
 │    │    ├── resources/    → Config files, application.yml, static files
 │    │
 │    ├── test/
 │         ├── java/         → Unit/integration tests
 │         ├── resources/    → Test configs/resources
 │
 ├── target/                 → Compiled output/build artifacts
 │
 ├── pom.xml                 → Maven project configuration
```

#### application.properties
- Located in src/main/resources folder -> Add configs here. (.env file in JS)
- *server.port=8000* -> Run app on port 8000
```Java
@RestController
class FunRestController{
	@Value("server.port")// Inject values from properties file into a class.
	private Integer portNumber;
}
```
##### SpringBoot properties
- **Core Properties**
```
# set different logging levels at different projects.

logging.level.org.springframework = DEBUG 
logging.level.org.hibernate = TRACE
logging.level.com.webapp = TRACE

# Log file name
logging.file.name = "logs.log"
logging.file.path = "~/appLogs"
```
- **Web Properties**
```md
# access your app at localhost:8000/my-app
server.port = 8000
server.servlet.context-path = /my-app  
# Session times out after 15 minutes
server.servlet.session.timeout = 15m 
```
- **Actuator Properties**
```md
management.endpoints.web.exposure.include = health,info
management.endpoints.web.exposure.exclude = beans
management.endpoints.web.base-path = /actuator/myapp
management.info.env.enabled = true
info.app.name = First SpringBoot Application
info.app.description = My first app on java
info.version = 1.0.0
```
- **Security Properties**
```
spring.security.user.name=<userName>
spring.security.user.password=<password>
```
- **Data Properties**
```
spring.datasource.url = jdbc:mysql://localhost:3306/ecommerce
spring.datasource.username=<userName>
spring.datasource.password=<password>
```
### SpringBoot Starters
- Curated list of maven dependencies by Spring Dev team to start your project.
- You get it from **Spring Initializer** - https://start.spring.io
- For webapps you use *Spring Web*.
#### SpringBoot Java Parent
- Collection of dependencies you need all grouped into one.
### SpringBoot Devtools
- Use to automatically restart application when code is updated.
- Add this to pom.xml and refresh Maven page.
```XML  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-devtools</artifactId>  
</dependency>
```
### SpringBoot Actuator
- Exposes REST endpoints -> health and monitoring status of your application.
- **/actuator/help** - {"groups":["liveness","readiness"],"status":"UP"}
- **/actuator/info** - Setup using the following properties. Returns the info object. {"app":{"name":"...","desc":"..."},"version":"1.0.0"}
```
management.endpoints.web.exposure.include = health,info
management.endpoints.web.exposure.exclude = beans
management.info.env.enabled = true
info.app.name = First SpringBoot Application
info.app.description = My first app on java
info.version = 1.0.0
```
- **/actuator/beans** - Returns all the registered beans.
- **/actuator/mappings** - Gives all the rest endpoints that are exposed.
#### Security
- Add - 
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-security</artifactId>  
</dependency>
```
- All the actuator endpoints that are exposed need security.
- By default username = user, password is printed on the console when you run the app.
```
spring.security.user.name=<userName>
spring.security.user.password=<password>
```
### Running from Commandline
```
// generate jar file
./mvnw package 
java -jar myApp.jar

// Use maven file
./mvnw springboot:run
```

## Spring Core
### Inversion of Control, Dependency Injection
- Spring Container has 2 responsibilities
	- **Inversion of Control** - App outsources object creation to Spring which acts like a factory.
	- **Dependency Injection** - Adding the dependencies of the requested object
- **Autowiring** - Spring will look for the class/interface that matches -> dependency is autowired.
#### Dependency Injection
- **Constructor Injection** - Used for required dependencies.
- **Setter Injection** - Used for optional dependencies when app can produce results even if dependency is absent.
#### Spring Beans
- **Bean** - Java class which is managed by Spring. Marked with **@Component**.
- **@Component** - Makes it available for DI.
- **@Autowired** - For constructors/setters where you need the bean, you add the **@Autowired** annotation.
#### Constructor Injection
- Any Spring Application Class needs **@SpringBootApplication** annotation.
- **@SpringBootApplication** is composed of 
	- *@EnableAutoConfiguration* - Enables Spring Boots auto configuration support.
	- *@ComponentScan* - Scans for components in the current package and sub packages recursively.
	- *@Configuration* - Able to register extra beans with @Bean or other import config classes.
- Component Scan only scans the package where the main file is located.
- If you want other packages to be scanned include them in @SpringBootApplication definition
```Java
@SpringBootApplication(scanBasePackage={"com.resource.util","edu.pack.support"})
public class mySpringApp{
}
```
#### Setter Injection
- Any method can be used to assign a bean dependency. This is called setter injection
- Not used anywhere realistically.
#### Field Injection
- Not used anywhere -> Directly set value to a field.
#### @Qualifier
- Suppose you are using an interface and you have multiple implementation.
- You need to provide Spring with a identifier to which implementation you actually need.
- If you need all the implementations, use List\<InterfaceName> as the dependency.
#### @Primary
- multiple beans implement the same interface and you want Spring to choose one as the **default** bean.
- You can mark only one implementation as primary.
#### @Lazy
- All beans are created before the app starts -> All beans even if they are not referenced.
- Such a Component will not be initialized if it is not referenced in the app.
- If you want to do it for all 
```
spring.main.lazy-initialization=true
```
- **Disadvantages** - 
	- RestControllers will not be initialized unless called. 
	- Config faults will not be identified.

### Example
```Java
@RestController  
public class AppController {  
    private Coach coach;  
    @Autowired
    private Player player;  //Field Injection
    private Physio physio;  
    private Subject sub1;  
    private Subject sub2;  
    private List<Subject> subjectList;  
    @Autowired  
    public AppController(Coach coach, Player player, @Qualifier("physics")Subject subject, List<Subject> subjectList, Subject sub2){  
        this.coach = coach;  
        this.player = player;  
        this.sub1 = subject;  
        this.subjectList = subjectList;  //Constructor Injection
        this.sub2 = sub2; // Picks the one with @Primary
    }  
    @Autowired  
    public void randomMethod(Physio physio){  
        this.physio = physio;  // Setter Injection
    }  
    @GetMapping("/advice")  
    public String getAdvice(){  
        return coach.getDailyWorkout();  
    }  
    @GetMapping("/getPhysio")  
    public String getPhysio(){  
        return physio.getPhysio();  
    }  
    @GetMapping("/getPlayer")  
    public String getPlayer(){  
        return player.getSport();  
    }  
    @GetMapping("/subject1")  
    public String getSub1(){  
        return sub1.getName();  
    }  
    @GetMapping("/getAllSubjects")  
    public String getSubjects(){  
        return subjectList.stream().map(s->s.getName()).collect(Collectors.joining(", "));  
    }  
}
```
### Bean Definition using Java Source Code
- We can use *@Component* annotation when we have the source code(class) available.
- If we want to use third party apps, we just have the jar file and the class name.
- Here we need to use **@Configuration** and **@Bean** annotations.
- **Note**- *In Qualifier, you need to pass the Bean method name.*
- *Or you can pass the qualifier name in the Bean annotation.*
```Java
@Configuration
public class DocumentsConfig{
	@Bean("s3Client")
	public S3Client remoteClient(){
		S3Client client = S3Client.builder().region(Region.US_East_1).credentialsProvider(provider).build();
		return client;
	}
}
```

### Beans
#### Scope
 - Defined using **@Scope**
```Java
@Scope(ConfigurabeBeanFactory.SCOPE_PROTOTYPE);
```
- **singleton** - *Default*, Creates a single instance which is shared.
- **prototype** - Creates 1 instance per container request.
- **request** - Scoped to a HTTP web request. Only for webapps.
- **session** - Scoped to a HTTP session. Only for webapps.
- **application** - Scoped to webApp ServletContext. ONly for webapps.
- **websocket** - Scoped to websocket only for webapps.
#### Lifecycle
```
Spring Starts -> Bean Definition Loaded -> Bean Object Created (Instantiation) 
-> Dependencies Injected
-> @PostConstruct / init() Called
-> Bean Ready For Use -> Application Running -> Container Shutdown
-> @PreDestroy / destroy() Called -> Bean Removed
```
- **@PostConstruct** - Runs after bean is injected.
- **@PreDestroy** - Runs after container shutdown, when bean is ready to be removed.
## Hibernate/JPA Crud
- **Hibernate** - ORM for Java -> Used to store Java objects to DB.
- **JPA** - Jakarta Persistence API -> Defines a set of interfaces. Hibernate implements JPA. 
	- Internally it uses JDBC.
	- It handles implementations of JDBC, db conn, password etc on its own.
	- We use the interfaces provided in the JPA while ORMs implement and pass their implementation to us.
```Java
Student student = new Student("John","Doe","john.doe@gmail.com");
entityManager.persist(student); // Saves the object
// Find object
int studId = 1;
student = entityManager.find(Student.class, studId); 
// Find list of all objects
TypedQuery<Student> query = entityManager.createQuery("FROM Student", Student.class)
List<Student> studentList = query.getResultList();
// Update Student
student.setName("Jane");
entityManager.merge(student);
// Delete Student
entityManager.remove(student);
```
### Setup
- Dependencies required - 
	- **MySQL Connector** - mysql-connector-j
	- **Spring Data** - spring-boot-starter-data-jpa
- In application.properties - 
```
spring.datasource.url = jdbc:mysql://localhost:3306/ecommerce
spring.datasource.username = user
spring.datasource.password = myuserdb
```
### Using JPA
#### Entities
- Each table has a corresponding entity class, with the **@Entity** annotation.
- Each entity must have a **public or protected no arguments Consturctor**.
- **@Table(name="")** - Maps the entity with the name of the table.
- **@Column(name="")** - Maps attributes to columns. Can be skipped but then attr names = column names
- **@Id** - id column needs to be annotated with this.
	- **@GeneratedValue** - specifies which strategy does the DB use to generate the id of new objects.
		- AUTO - pick appropriate strategy based on DB
		- IDENTITY - ids are generated based on DB id column
		- SEQUENCE - Assign ids using a DB sequence
		- TABLE - Use a db table to assign ids
		- UUID - Assign PK using a UUID.
```Java
@Entity
@Table(name="student")
public class Student{
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="id")
	private int id;
	@Column(name="first_name")
	private String firstName;
}
```
#### EntityManager
- Actually cruds the objects to the table.
- Provides low level control. It can also help us write custom queries.
- **JpaRepository** - Alternative to EntityManager which provides greater abstraction. 
#### Data Access Objects
- DAO defines ways to access your entity.
- **DAO Interface** - You define all the methods you want to allow the application to access.
- **DAOImpl** - Implements the methods.
##### @Transaction
- You want your updates/saves to run in a txn. On Implementation of methods add this annotation.
##### @Repository
- Applied to DAOImpl to make Spring aware of the implementation. Subclass of @Component.
#### AutoCreate Tables
- Generates tables based on entities defined
```
spring.jpa.hibernate.ddl-auto=create
```
- Values of this property
	- none - No action
	- create - Drop existing table and create a new one.
	- create-drop - Drop existing table on application shutdown and create a new one on startup.
	- validate - Validate entities a gainst DB tables.
	- update - Update the db table against our entity.
#### Seeing SQL Logs
```
logging.level.org.hibernate.SQL = debug
logging.level.org.hibernate.orm.jdbc.bind = trace
```
## REST APIs
- You require Spring Web dependency.
- For each POST/PATCH api you need an Entity or a Data Transfer Object(POJO).
### RestController
- All rest apis are defined in a object marked with **@RestController**
- **@RequestMapping("/test")** - Specify the path under which we have these api.
### Databinding
- Conversion of JSON data to Java POJO
- **Jackson** - Project used for data binding in Spring. Calls setters/getters to do this.
	- JSON to POJO -> setters. POJO to JSON -> getters.
	- Sits between the client and your application. Converts incoming JSON to POJO and outgoing to JSON.
### Path Variables
- /api/student/{id}
- In the method, define a variable with **@PathVariable**
### Exception Handling
- If any error occurs, we want to return a JSON response.
- **Error Class** - POJO, defines the structure of response you want to send.
- **Custom Exception** - simple class which extends RuntimeException
	- When the condition is violated, throw this exception.
```Java
public class StudentNotFoundException extends RuntimeException{
	public StudentNotFoundException(String message){
		super(message);
	}
}
```
- **Exception Handler** - to be added in the rest controller.
```Java
@ExceptionHandler
public ResponseEntity<StudentErrorResponse>  handleException(StudentNotFoundException e){
	StudentErrorResponse error = new StudentErrorResponse();
	error.setStatus(HttpStatus.NOT_FOUND.value());
	error.setMessage(e.getMessage());
	error.setTimestamp(System.currentTimeMillis());
	return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
}
```
#### Global Exception Handling
- Create a new exception handler class to handle all exceptions defined with **@ControllerAdvice**
### Service Layer
- Defined using - **@Service**
- Suppose you have multiple DAOs to pull your data from -> Generate a facade called the Service Layer.
- It contains all the daos and is responsible for fetching data from multiple daos.
- RestController just calls the service layer class methods.
- **@Transactional** - moved to service layer as it should manage transactions.
### Spring Data JPA
- Dont need to add DAOs. It gives us crud methods of DAOs.
- You create a new interface extending JPARepository<Entity, PK_dataType> and inject it in your service layer.
```Java
public interface EmployeeRepository extends JpaRepository<Employee, Integer> {
}
```
### Spring Data REST
- RestController code is boilerplate. We use the same methods - POST, GET, GETALL, PUT, PATCH and DELETE
- Just the entities change, basic CRUD implementation is the same.
- **Uses Spring Data JPA -> @JPARepository**
- Defines the basic crud at the plural form of the entity. Eg- Employee -> /employees
- You just need to add **spring-boot-starter-data-rest** as dependency.
##### API path
- *@RepositoryRestResource(path = "employees")* - You can specify the path in JPARepository interface.
##### Sort
```
http://localhost:8080/employees?sort=lastName
http://localhost:8080/employees?sort=firstName,desc
http://localhost:8080/employees?sort=firstName,lastName,asc
```
##### Spring Data Config
```
spring.data.rest.base-path="/apiPath"
spring.data.rest.default-page-size=20
spring.data.rest.max-page-size=20
```
### OpenAPI and Swagger
- **Springdoc** is an open source project which automatically generates API documentation.
- Swagger generates a UI for interacting with API.(No need for Postman)
- Add dependency. Get version from www.springdoc.org
```
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>...</version>
</dependency>
```
- Custom path - 
```
springdoc.swagger-ui.path=/swaggerUi/pathname
```
### DTOs and Validations
- For GET and POST calls that dont expect/return entities, we need to define DTOs
- **Data Transfer Object** - POJO with the fields you want to return.
- If you pass more than what the API expects, other properties are ignored.
#### Validation Annotations
- Add the following dependency - 
```
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-validation</artifactId>  
</dependency>
```
```Java
public class CreateUserRequestDTO {
    @NotBlank
    private String name;
    @Email
    private String email;
    @Size(min = 8)
    private String password;
}

@PostMapping("")
public User create(@Valid @RequestBody CreateUserRequestDTO dto) {
}
```
- Some annotations. Each one has an optional message parameter.
	- **@NotNull**
	- **@NotBlank** - string cannot be null/empty/whitespace 
	- **@NotEmpty** - collection/string cannot be empty      
	- **@Size(min,max)** - length/size constraints                
	- **@Min(1)** - minimum numeric value                  
	- **@Max(10)** - maximum numeric value                  
	- **@Positive**
	- **@PositiveOrZero**
	- **@Negative**
	- **@Email**
	- **@Pattern(regexp="")** - regex validation                       
	- **@Past** - date must be in past                   
	- **@Future** - date must be in future    
#### Custom Validation Annotation

## Spring Security
- **Authentication** - Check userId and password
- **Authorization** - Check for user roles
- Acts as a middleware between client and RestController.
- By default it protects all the apis with user, pwd generated in console.
- You can define custom user and password.
```
spring.security.user.name=<userName>
spring.security.user.password=<password>
```
- All security logic is defined in a **@Configuration** class.
- You can create extra beans here through methods using **@Bean** annotation
- To enable add dependency - *spring-boot-starter-security*
### In memory Users
```Java
@Configuration  
public class DemoSecurityConfig {  
    @Bean  
    public InMemoryUserDetailsManager userDetailsManager(){  
        UserDetails john = User.builder().username("John")  
                .password("{noop}password")  
                .roles("EMPLOYEE")  
                .build();  
        UserDetails mary = User.builder().username("Mary")  
                .password("{noop}password")  
                .roles("EMPLOYEE","MANAGER")  
                .build();  
        UserDetails susan = User.builder().username("Susan")  
                .password("{noop}password")  
                .roles("EMPLOYEE","MANAGER","ADMIN")  
                .build();  
        return new InMemoryUserDetailsManager(john,mary,susan);  
    }  
}
```
- **password** -> {type}yourPassword
	- type = **noop** -> Store plain text password
	- type = **bcrypt** -> store encrypted passwords Eg - {bcrypt}\<bcryptHash>
### DB Persisted Users
#### Using Default
- By default if you create tables named users and authorities as follows, spring will automatically fetch users.
- Authorities is the same as roles.
```MySQL
CREATE TABLE `users` (
  `username` varchar(50) NOT NULL,
  `password` varchar(50) NOT NULL,
  `enabled` tinyint NOT NULL,
  PRIMARY KEY (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

CREATE TABLE `authorities` (
  `username` varchar(50) NOT NULL,
  `authority` varchar(50) NOT NULL,
  UNIQUE KEY `authorities_idx_1` (`username`,`authority`),
  CONSTRAINT `authorities_ibfk_1` FOREIGN KEY (`username`) REFERENCES `users` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

INSERT INTO `users` 
VALUES 
('john','{noop}test123',1),
('mary','{noop}test123',1),
('susan','{noop}test123',1);

INSERT INTO `authorities` 
VALUES 
('john','ROLE_EMPLOYEE'),
('mary','ROLE_EMPLOYEE'),
('mary','ROLE_MANAGER'),
('susan','ROLE_EMPLOYEE'),
('susan','ROLE_MANAGER'),
('susan','ROLE_ADMIN');
```
#### Custom Tables
- You can use any table names and columns you want.
- You just need to tell Spring the SQL statements to fetch users by username and roles by username.
```Java
@Bean  
public UserDetailsManager userDetailsManager(DataSource dataSource){  
    JdbcUserDetailsManager jdbcUserDetailsManager = new JdbcUserDetailsManager(dataSource);  
    jdbcUserDetailsManager.setUsersByUsernameQuery("select user_id, pw, active from members where user_id = ?");  
    jdbcUserDetailsManager.setAuthoritiesByUsernameQuery("select user_id, role from roles where user_id = ?");  
    return jdbcUserDetailsManager;  
}
```
### Restricting Access based on roles
- Add this to you *@Configuration* class
```Java
@Bean  
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception{  
    http.authorizeHttpRequests(configurer->  
        configurer.requestMatchers(HttpMethod.GET, "/api/employees").hasRole("EMPLOYEE")  
		.requestMatchers(HttpMethod.GET, "/api/employees/**").hasRole("EMPLOYEE") // Matches on all subpaths of /employees 
                .requestMatchers(HttpMethod.POST, "/api/employees").hasRole("MANAGER")  
                .requestMatchers(HttpMethod.PUT, "/api/employees").hasRole("MANAGER")  
                .requestMatchers(HttpMethod.PATCH, "/api/employees/**").hasRole("MANAGER")  
                .requestMatchers(HttpMethod.DELETE, "/api/employees/**").hasRole("ADMIN")  
    );  
    http.httpBasic(Customizer.withDefaults());  
    http.csrf(csrf->csrf.disable());  
    return http.build();  
}

```
## Hibernate Advanced Mapping
- Mappings - How we map tables with objects.
	- **one-to-one**
	- **one-to-many**
	- **many-to-many**
- **Unidirectional** -> One Table has ref to the other table.
  **Bidirectional** -> Both have references to each other. 
- **Cascading Delete** - Delete all entries that depend on this table.
- **Eager vs Lazy** - If you fetch from a table do you fetch the details together with it?
### Entity Lifecycle
- These are the lifecycle stages of an entity object. Hibernate monitors every object and makes changes to db.
	- **Transient** - Object created. Not persisted to entityManager.
	- **Managed** - Hibernate watches and syncs any changes to DB.
	- **Detached** - Removed from watch. Any changes will not sync up with DB.
	- **Remove** - Row is marked for deletion.
- Operations - *persist, remove, refresh, detach* and *merge*
### Cascading
- Cascade operation is performed to the object and any related ones at the same time.
- By default no operation is cascaded.
- Types - 
	- **PRESIST** - If entity is persisted/saved, related entities will also be saved.
	- **REMOVE**
	- **REFRESH**
	- **DETACH**
	- **MERGE** - Move from detach to manage.
	- **ALL**
### Bidirectional Mapping
- You define one FK by using @JoinColumn(name="columnName")
- The other FK can be defined using @OneToOne(mappedBy="instructorDetail") -> name of the property in 1st class.
### Lazy Vs Eager
- Each mapping has a default fetch parameter.
	- **@OneToOne(fetch=)** - FetchType.EAGER
	- **@OneToMany(fetch=)** - FetchType.LAZY
	- **@ManyToOne(fetch=)** - FetchType.EAGER
	- **@ManytoMany(fetch=)** - FetchType.LAZY
- If you have a LAZY fetch, you cannot directly access the linked entity -> You need to add a new method to fetch the entity.
- Or you can use a Typed query with JOIN FETCH -> Eager load in a method if you want eager loaded data.
```

```
![[WhatsApp Image 2026-05-13 at 18.19.52.jpeg|500]]
## Aspect Oriented Programming
- **Aspect** - encapsulate a cross cutting logic -> A logic that executes in all the code that executes for all requests.
```
Main App -> AOP Proxy(does Logging,Security) -> Target Object.
```
- Think of it as a middleware
- Uses - *Logging, Security, Transactions, Exception Handling*
- **Pros** - Reusable modules, No code replication.
- **Cons** - Performance

 ![[WhatsApp Image 2026-05-14 at 18.02.25.jpeg|500]]


## Spring MVC
- Old way of rendering frontend from the server - 
	- **Model** - It is the application state. This is modified by the controller and rendered by the view.
	- **View** - Code that renders data on frontend.
	- **Controller** - It is the backend code that interacts with the model and sends data to the view.
```
Client (React/Postman/Mobile) -> DispatcherServlet -> Controller 
-> Service -> Repository -> Database
```
- **DispatcherServlet** - This is the frontend controller. All client requests come here.
- **Controller** - Handle HTTP requests -> Interacts with the model layer.
### Terms
- **Aspect** - Module of code for cross cutting concern.
- **Advice** - What action is to be taken
- **Join Point** - When to Apply the action.
- **Pointcut** - Expression for when to apply the action.
### Types of Advice
- **@Before** - Before the method.
- **After Finally** - Finally after the method executes error or success.
- **After Returning** - After the method is successful.
- **After Throwing** - After the method throws error. 
- **Around** - Runs before and after the code.
### Examples
- Execute before any method with signature - **public void addAccount()** 
```Java
@Before("execution(public void addAccount())")  
public void beforeAddAccountAdvice(){  
    System.out.println("\n ========> Before Advice executed");  
}
```
### Pointcut Expression
```Java
execution(modifiers-pattern? return-type-pattern declaring-type-pattern? method-name-pattern(param-pattern) throws-pattern)

?-> optional

-- Before addAcount method in AccountDAO
@Before("execution(public void com.webApp.aop.dao.AccoutDAO.addAccount())")

-- Before and addAccount method
@Before("execution(public void addAccount())")

-- Using wildcards - Any method starting with *
@Before("execution(public void add*())")

-- Wildcard in return type
@Before("execution(public * process*())")

@Before("execution(* add*())")
@Before("execution(void add*()) || execution(boolean add*())")
@Before("execution(void com.webApp.aop.*.*.add*()) || execution(boolean add*())")

-- On parameter types
() -> 0 parameters
(*) -> 1 parameter of any type
(..) -> 0 or more parameters of any type

```
#### Pointcut Declaration
- Define pointcuts which can be reused.
- Define pointcut as follows and pass it in other @Before annotations
```Java
@Pointcut("execution(void add*(com.webApp.aop.Account, ..))")  
public void matchParams3(){}

@Before("matchParams3()")  
public void newMethod(){  
System.out.println("New Advice with duplicated code");  
}
```
#### Chaining Pointcut Expressions
```Java
@Before("execution(void com.webApp.aop.*.*.add*()) || execution(boolean add*())")
@Before("execution(void com.webApp.aop.*.*.add*()) || !execution(boolean add*())")
@Before("execution(void com.webApp.aop.*.*.add*()) && !execution(boolean add*())")
```
#### Ordering Aspects
- Refactor methods and club into different Aspects.
- Add @Order(x) annotations to order the execution of different aspects.
- Lower values of x have a higher precedence.
```Java
@Aspect  
@Component  
@Order(1)
public class DemoLoggingAspect {
```

### JoinPoint
- Has information about the method we are about to work with.
```Java  
@Before("mypointCut()")  
public void dupPointCut2(JoinPoint joinPoint){  
    System.out.println("Duplicated pointCut 2");  
    Signature signature = joinPoint.getSignature();  
    System.out.println("Method - "+signature);  
    
}
```
### @AfterReturning
```Java 
@AfterReturning(pointcut = "execution(* findAccounts(..))",  
                returning = "result")  
public void afterReturningMethod(JoinPoint joinPoint, List<Account> result){  
    System.out.println("After returning call - ");  
    System.out.println(joinPoint.getSignature());  
    System.out.println(result);  
}
```
### @AfterThrowing
- Runs before the catch block.
```Java 
@AfterThrowing(pointcut = "execution(* findAccounts(..))", throwing= "exc")  
public void afterThrowinMethod(JoinPoint joinPoint, Exception exc){  
    System.out.println("After returning call - ");  
    System.out.println(joinPoint.getSignature());  
    System.out.println("--> " + exc.getMessage());  
}
```
### @After
- Runs after method completion eiter success or failure. Works like finally.
```Java

```
### Weaving
- Connecting aspects to target objects to create an advised object.
- Types - **CompileTime, Load-time** and **Runtime**
- **Runtime is the slowest**
### Spring AOP vs Aspect J
- Inbuilt support for AOP. Provides runtime AOP.
- AspectJ provides full support for AOP.
- You create a new class with @Aspect and @Component annotation and add methods.


| Section                          | Action      | Priority | Notes                      |
| -------------------------------- | ----------- | -------- | -------------------------- |
| Advanced Mapping Cheat Sheet     | Watch       | Medium   | Nice revision              |
| AOP Business Problem / Concepts  | Watch       | High     | Understand why AOP exists  |
| Spring AOP vs AspectJ            | Skim        | Medium   | Conceptual                 |
| @Before / Pointcuts              | Watch       | Medium   | Basic understanding enough |
| Ordering Aspects                 | Skim        | Low      | Not critical               |
| JoinPoints                       | Skim        | Medium   | Useful awareness           |
| AfterReturning / Around Advice   | Watch       | Medium   | Understand proxies         |
| Around Advice Exception Handling | Skim        | Medium   | Awareness sufficient       |
| AOP + MVC CRUD Integration       | Skip Mostly | Low      | Too implementation-heavy   |
