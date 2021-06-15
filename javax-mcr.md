@WebMvcTest
@MockBean

```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.hamcrest.Matchers.equalTo;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
```

```java
@Test
void testListEmployees() throws Exception {
    when(employeesService.listEmployees(any())).thenReturn(List.of(
            new EmployeeDto(1L, "John Doe"),
            new EmployeeDto(2L, "Jane Doe")
    ));
    mockMvc.perform(get("/api/employees"))
      .andDo(print())
            .andExpect(status().isOk())        
            .andExpect(jsonPath("$[0].name", equalTo("John Doe")));
}
```

# Teljes alkalmazás

```java
@Test
void testListEmployees() throws Exception {
  mockMvc.perform(get("/api/employees"))
    .andExpect(status().isOk())
    .andDo(print())
    .andExpect(
      jsonPath("$[0].name", equalTo("John Doe")));
}
```

# RestTemplate

```java
@SpringBootTest(webEnvironment =
    SpringBootTest.WebEnvironment.RANDOM_PORT)
```


```java
@Test
void testListEmployees() {
    restTemplate.exchange("/api/employees",
      HttpMethod.POST,
      new CreateEmployeeCommand(),
      new ParameterizedTypeReference<List<EmployeeDto>>(){})
    .getBody();



  List<EmployeeDto> employees = 
    restTemplate.exchange("/api/employees",
      HttpMethod.GET,
      null,
      new ParameterizedTypeReference<List<EmployeeDto>>(){})
    .getBody();
  assertThat(employees)
          .extracting(EmployeeDto::getName)
          .containsExactly("John Doe", "Jane Doe");
}
```

```java
deleteAll
```

# Swagger UI

```xml
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-ui</artifactId>
  <version>${springdoc-openapi-ui.version}</version>
</dependency>
```

```java
@Schema(description="name of the employee", example = "John Doe") 
@Tag( name = "Operations on employees")
@Operation(summary = "Find employee by id",
    description = "Find employee by id.")
@ApiResponse(responseCode = "404",
    description = "Employee not found")
```

# REST Assured

```xml
<dependency>
  <groupId>io.rest-assured</groupId>
  <artifactId>json-path</artifactId>
  <version>${rest-assured.version}</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>io.rest-assured</groupId>
  <artifactId>xml-path</artifactId>
  <version>${rest-assured.version}</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>io.rest-assured</groupId>
  <artifactId>rest-assured</artifactId>
  <version>${rest-assured.version}</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>io.rest-assured</groupId>
  <artifactId>spring-mock-mvc</artifactId>
  <version>${rest-assured.version}</version>
  <scope>test</scope>
</dependency>
```

```java
import io.restassured.http.ContentType;
import io.restassured.module.mockmvc.RestAssuredMockMvc;
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;
import static io.restassured.module.mockmvc.RestAssuredMockMvc.*;
import static org.hamcrest.Matchers.equalTo;
```

```java
@SpringBootTest
@AutoConfigureMockMvc
public class EmployeesControllerRestAssuredIT {

    @Autowired
    MockMvc mockMvc;

    @BeforeEach
    void init() {
        RestAssuredMockMvc.requestSpecification =
                given().contentType(ContentType.JSON)
                .accept(ContentType.JSON);
        RestAssuredMockMvc.mockMvc(mockMvc);
    }

    @Test
    void testCreateThenListEmployee() {
        with()
                .body(new CreateEmployeeCommand("John Doe"))
                        .when()
        .post("/api/employees")
        .then()
                .body("name", equalTo("John Doe"));

        with()
        .get("/api/employees")
                .then()
                .body("[0].name", equalTo("John Doe"))
        .log();

        with()
                .get("/api/employees")
                .then()
                .statusCode(200)
                .body("size()", equalTo(1))
        .log();
    }
}
```

# Rest Assured séma validáció

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>json-schema-validator</artifactId>
    <scope>test</scope>
</dependency>
```


```json
"additionalProperties": false
```

```java
.body(matchesJsonSchemaInClasspath("employee-dto.json"))
```

# Content negotiation

```java
@RequestMapping(value = "/api/employees",
  produces = {MediaType.APPLICATION_JSON_VALUE,
    MediaType.APPLICATION_XML_VALUE})
```

```xml
<dependency>
  <groupId>org.glassfish.jaxb</groupId>
  <artifactId>jaxb-runtime</artifactId>
</dependency>
```

```java
@XmlRootElement
```

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@XmlRootElement(name = "employees")
@XmlAccessorType(XmlAccessType.FIELD)
public class EmployeesDto {
    @XmlElement(name = "employee")
    private List<EmployeeDto> employees;
}
```

# Validáció

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

```java
@NotNull(message = "Name can not be null")
```

```java
@Valid
```

RFC 7807

```java
@ExceptionHandler({MethodArgumentNotValidException.class})
public ResponseEntity<Problem> handleValidationError(MethodArgumentNotValidException e) {
    List<Violation> violations = e.getBindingResult().getFieldErrors().stream()
            .map((FieldError fe) -> new Violation(fe.getField(), fe.getDefaultMessage()))
            .collect(Collectors.toList());
    Problem problem = Problem.builder()
            .withType(URI.create("employees/validation-error"))
            .withTitle("Validation error")
            .withStatus(Status.BAD_REQUEST)
            .withDetail(e.getMessage())
            .with("violations", violations)
            .build();
    return ResponseEntity
      .status(HttpStatus.BAD_REQUEST)
      .contentType(MediaType.APPLICATION_PROBLEM_JSON)
      .body(problem);
}
```

# Saját validáció

```java
@Constraint(validatedBy = NameValidator.class)
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
public @interface Name {
    String message() default "Invalid name";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    int maxLength() default 50;
}
```

```java
public class NameValidator implements ConstraintValidator<Name, String> {
  private int maxLength;
  @Override
  public boolean isValid(String value, ConstraintValidatorContext context) {
    return value != null &&
      !value.isBlank() &&
      value.length() > 2 && 
      value.length() <= maxLength && 
      Character.isUpperCase(value.charAt(0));
  }
  @Override
  public void initialize(Name constraintAnnotation) {
      maxLength = constraintAnnotation.maxLength();
  }
}
```

# Spring Boot konfiguráció

```java
private String hello;
  public HelloService(@Value("${employees.hello}") String hello) {
    this.hello = hello;
  }
```

```java
@ConfigurationProperties(prefix = "employees")
@Data
public class HelloProperties {
    private String hello;
}
```

```java
@EnableConfigurationProperties(AcmeProperties.class)
```

* Parancssorból
* `set EMPLOYEES_HELLO="Hello from Environment variable"`
* `java -Demployees.hello=`
* `docker build -t employees .`
* `docker run -d -p 8080_8080 -e EMPLOYEES_HELLO="Hello from Docker" employees`

# Naplózás

```java
private static final org.slf4j.Logger log =
  org.slf4j.LoggerFactory.getLogger(LogExample.class);
@Slf4j
log.info("Employee has been created");
log.debug("Employee has been created with name {}",
  employee.getName());
```

```
logging.level.training = debug
```

# Spring JdbcTemplate

`org.springframework.boot:spring-boot-starter-jdbc` függőség
Embedded adatbázis támogatás, automatikus DataSource konfiguráció
Pl H2: `com.h2database:h2`

```
List<Employee> employees = jdbcTemplate.query(
  "select id, emp_name from employees",
  this::convertEmployee);
Employee employee =
  jdbcTemplate.queryForObject(
    "select id, emp_name from employees where id = ?",
    this::convertEmployee,
    id);
private Employee convertEmployee(ResultSet resultSet, int i)
    throws SQLException {
  long id = resultSet.getLong("id");
  String name = resultSet.getString("emp_name");
  Employee employee = new Employee(id, name);
  return employee;
}
```

```
KeyHolder keyHolder = new GeneratedKeyHolder();
jdbcTemplate.update(
  con -> {
      PreparedStatement ps =
        con.prepareStatement("insert into employees(emp_name) values (?)",
          Statement.RETURN_GENERATED_KEYS);
      ps.setString(1, employee.getName());
      return ps;
  }, keyHolder);
employee.setId(keyHolder.getKey().longValue());
```

```
jdbcTemplate.update(
  "update employees set emp_name = ? where id = ?", "John Doe", 1);
jdbcTemplate.update(
  "delete from employees where id = ?", 1);
```

```
@Component
public class DbInitializer implements CommandLineRunner {
  @Autowired
  private JdbcTemplate jdbcTemplate;
  @Override
  public void run(String... args) throws Exception {
    jdbcTemplate.execute("create table employees " +
      "(id bigint auto_increment, emp_name varchar(255), " +
      "primary key (id))");
    jdbcTemplate.execute(
      "insert into employees(emp_name) values ('John Doe')");
    jdbcTemplate.execute(
      "insert into employees(emp_name) values ('Jack Doe')");
  }
}
```

Developer Tools esetén elérhető webes konzol a `/h2-console` címen

# Spring Data JPA

```
org.springframework.boot:spring-boot-starter-data-jpa
```

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    @Column(name = "emp_name")
    private String name;
    public Employee(String name) {
        this.name = name;
    }
}
```

```java
public interface EmployeesRepository extends JpaRepository<Employee, Long> {
    @Query("select e from Employee e where upper(e.name) like upper(:name)")
    List<Employee> findAllByPrefix(String name);
}
```

* `@Transactional` alkalmazása a service rétegben
* `application.properties`: `spring.jpa.show-sql=true`

# MariaDB

A MariaDB a következő Docker paranccsal indítható:

```shell
docker run 
  -d
  -e MYSQL_DATABASE=employees
  -e MYSQL_USER=employees
  -e MYSQL_PASSWORD=employees
  -e MYSQL_ALLOW_EMPTY_PASSWORD=yes
  -p 3306:3306
  --name employees-mariadb
  mariadb
```

H2 helyett

```xml
<dependency>
  <groupId>org.mariadb.jdbc</groupId>
  <artifactId>mariadb-java-client</artifactId>
  <scope>runtime</scope>
</dependency>
```

Az `application.properties` állományban:

```properties
spring.datasource.url=jdbc:mariadb://localhost/employees
spring.datasource.username=employees
spring.datasource.password=employees

spring.jpa.hibernate.ddl-auto=create-drop
```

Create, list

MariaDB DataSource

```
select * from employees;
```

# Integrációs tesztelés

```java
@DataJpaTest
public class EmployeesRepositoryIT {

  @Autowired
  EmployeesRepository employeesRepository;

  @Test
  void testPersist() {
    Employee employee = new Employee("John Doe");
    employeesRepository.save(employee);
    List<Employee> employees =
      employeesRepository.findAll();
    
    assertThat(employees)
      .extracting(Employee::getName)
      .containsExactly("John Doe");
  }
}
```

```shell
docker run 
  -d
  -e MYSQL_DATABASE=employees
  -e MYSQL_USER=employees
  -e MYSQL_PASSWORD=employees
  -e MYSQL_ALLOW_EMPTY_PASSWORD=yes
  -p 3307:3306
  --name employees-test-mariadb
  mariadb
```

* `application.properties` a test ágra, port átírása

# Állapot inicializálása

* `EmployeesControllerRestTemplateIT.testListEmployees()` futtatása
* `application.properties`-ben `create-drop` helyett `create`
* `deleteAll()` törlése

```java
@Sql(statements = "delete from employees")
```

# H2 integrációs teszteléshez

```xml
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <scope>test</scope>
</dependency>
```

```
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=sa
```

# Alkalmazás futtatása Dockerben MariaDB-vel

```
docker network create --driver bridge employees-net
```

```shell
docker run 
  -d
  -e MYSQL_DATABASE=employees
  -e MYSQL_USER=employees
  -e MYSQL_PASSWORD=employees
  -e MYSQL_ALLOW_EMPTY_PASSWORD=yes
  -p 3308:3306
  --name employees-net-mariadb
  mariadb
```

```shell
docker stop employees
docker rm employees
docker run
    -d  
    -e SPRING_DATASOURCE_URL=jdbc:mariadb://employees-net-mariadb/employees
    -p 8080:8080
     --network employees-net
    --name employees
    employees
```

# Alkalmazás futtatása Docker Compose-zal

* `Dockerfile`

```
RUN apt update \
     && apt-get install wget \
     && apt-get install -y netcat \
     && wget https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh \
     && chmod +x ./wait-for-it.sh
```

* `docker-compose.yml`

```yaml
version: '3'

services:
  employees-mariadb:
    image: mariadb
    restart: always
    ports:
      - '3308:3306'
    environment:
      MYSQL_DATABASE: 'employees'
      MYSQL_USER: 'employees'
      MYSQL_PASSWORD: 'employees'
      MYSQL_ALLOW_EMPTY_PASSWORD: 'yes'
    healthcheck:
      test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"]
      timeout: 20s
      retries: 10

  employees-app:
    image: employees
    restart: always
    ports:
      - '8080:8080'
    depends_on:
      - employees-mariadb
    environment:
      SPRING_DATASOURCE_URL: 'jdbc:mariadb://employees-mariadb/employees'
    entrypoint: ['./wait-for-it.sh', '-t', '120', 'employees-mariadb:3306', '--', 'java', 'org.springframework.boot.loader.JarLauncher']
```

# Flyway

```xml
<dependency>
  <groupId>org.flywaydb</groupId>
  <artifactId>flyway-core</artifactId>
</dependency>
```

```properties
spring.jpa.hibernate.ddl-auto=none
```

* `src/resources/db/migration/V1__employees.sql`

```sql
create table employees (id bigint auto_increment,
  emp_name varchar(255), primary key (id));
insert into employees (emp_name) values ('John Doe');
insert into employees (emp_name) values ('Jack Doe');
```

* Táblák törlése

# MongoDB

```shell
docker run -d -p27017:27017 --name employees-mongo mongo
```

* project copy: `employees-mongo`, `pom.xml`
* Törlés: `spring-boot-starter-jdbc`, `spring-boot-starter-data-jpa`, `h2`, `mariadb-java-client`, `postgresql`, `flyway-core`, `liquibase-core`.

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

Az `application.properties` fájlban:

```
spring.data.mongodb.database = employees
```

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Document("employees")
public class Employee {

    @Id
    private String id;

    private String name;

    public Employee(String name) {
        this.name = name;
    }
}
```

Módosítsuk a repository-t:

```java
public interface EmployeesRepository extends MongoRepository<Employee, String> {
    
}
```

A Dto-kban, Controllerben, Service metódus paraméterekben mindenütt, ahol `Long` típusú
azonosítót adtunk tovább, ki kell cserélni `String` típusra.

Az `EmployeesService.updateEmployee()` metódusát is módosítani kell, hogy legyen mentés:

```java
public EmployeeDto updateEmployee(long id, UpdateEmployeeCommand command) {
    Employee employee = repository.findById(id)
      .orElseThrow(() -> new IllegalArgumentException("employee not found"));
    employee.setName(command.getName());
    employeesRepository.save(employee);
    return modelMapper.map(employee, EmployeeDto.class);
}
```

* MongoDB connection, Database `employees`

```javascript
db.employees.find()

db.employees.insertOne({"name": "John Doe"})

db.employees.findOne({'_id': ObjectId('60780cf974bc5648cf220a96')})

db.employees.deleteOne({'_id': ObjectId('60780cf974bc5648cf220a96')})
```

