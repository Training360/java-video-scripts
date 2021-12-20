# Problem

```xml
<dependency>
   <groupId>org.zalando</groupId>
   <artifactId>problem-spring-web-starter</artifactId>
   <version>0.26.2</version>
</dependency>
```

```java
public class EmployeeNotFoundException extends AbstractThrowableProblem {

    private static final URI TYPE
            = URI.create("employees/employee-not-found");

    public EmployeeNotFoundException(Long id) {
        super(
                TYPE,
                "Not found",
                Status.NOT_FOUND,
                String.format("Employee with id '%d' not found", id));
    }
}
```


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
* Létrehozás, listázás http fájlból
* MongoDB connection, Database `employees`

```javascript
db.employees.find()

db.employees.insertOne({"name": "John Doe"})

db.employees.findOne({'_id': ObjectId('60780cf974bc5648cf220a96')})

db.employees.deleteOne({'_id': ObjectId('60780cf974bc5648cf220a96')})
```

# KeyCloak indítása

```shell
docker run -e KEYCLOAK_USER=root -e KEYCLOAK_PASSWORD=root -p 8081:8080
  --name keycloak jboss/keycloak
```

* Létre kell hozni egy Realm-et (`EmployeesRealm`)
* Létre kell hozni egy klienst, amihez meg kell adni annak azonosítóját, <br /> és hogy milyen url-en érhető el (`employees-app`)
* Létre kell hozni a szerepköröket (`employees_app_user`)
* Létre kell hozni egy felhasználót (a _Email Verified_ legyen _On_ értéken, hogy be lehessen vele jelentkezni), beállítani a jelszavát (a _Temporary_ értéke legyen _Off_, hogy ne kelljen jelszót módosítani), <br /> valamint hozzáadni a szerepkört (`johndoe`)

```
http://localhost:8081/auth/realms/EmployeesRealm/.well-known/openid-configuration
http://localhost:8081/auth/realms/EmployeesRealm/protocol/openid-connect/certs
```

```http
POST http://localhost:8081/auth/realms/Employees/protocol/openid-connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=password&client_id=employees-app&username=johndoe&password=johndoe
```

* Postmanből: Be kell írni egy létező scope-ot (pl. `profile`), mert üreset nem tud értelmezni

# KeyCloak integráció

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.keycloak.bom</groupId>
      <artifactId>keycloak-adapter-bom</artifactId>
      <version>14.0.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

```xml
<dependency>
  <groupId>org.keycloak</groupId>
  <artifactId>keycloak-spring-boot-starter</artifactId>
</dependency>
```


```properties
keycloak.auth-server-url=http://localhost:8081/auth
keycloak.realm=Employees
keycloak.resource=employees-app
keycloak.bearer-only=true

keycloak.security-constraints[0].authRoles[0]=employees_app_user
keycloak.security-constraints[0].securityCollections[0].patterns[0]=/*

keycloak.principal-attribute=preferred_username
```

```http
GET http://localhost:8080/api/employees
Accept: application/json
Authorization: bearer eyJ...
```

* Postmannel

```java
@GetMapping
public List<EmployeeDto> employees(Principal principal) {
    log.info("Logged in user: {}", principal.getName());
    return employeeService.listEmployees();
}
```

# RestTemplate

```shell
docker run -d -p 8081:8080 --name my-addresses training360/addresses
```

* Swagger
* `AddressDto`, `city`, `address`

```java
@Service
@Slf4j
public class AddressesGateway {

    private final RestTemplate restTemplate;

    private String url;

    public AddressesGateway(RestTemplateBuilder builder, 
            @Value("${employees.addresses.url}") String url) {
        restTemplate = builder.build();
        this.url = url;
    }

    public AddressDto findAddressByName(String name) {
        log.debug("Get address from Addresses application");
        return restTemplate.getForObject(url, AddressDto.class, name);
    }
}
```

```properties
employees.addresses.url = http://localhost:8081/api/addresses?name={name}
```

* `/api/employees/1/address`

## RestTemplate integrációs tesztelés 

```java
package training.employees;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.client.RestClientTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.client.MockRestServiceServer;

import static org.hamcrest.Matchers.startsWith;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.springframework.test.web.client.match.MockRestRequestMatchers.queryParam;
import static org.springframework.test.web.client.match.MockRestRequestMatchers.requestTo;
import static org.springframework.test.web.client.response.MockRestResponseCreators.withSuccess;

@RestClientTest(value = AddressesGateway.class, properties = "employees.addresses.url = http://localhost:8080/api/addresses?name={name}")
public class EventStoreGatewayRestTemplateTestIT {

    @Autowired
    AddressesGateway addressesGateway;

    @Autowired
    MockRestServiceServer server;

    @Autowired
    ObjectMapper objectMapper;

    @Test
    void testFindAddressByName() throws JsonProcessingException {
        String json = objectMapper.writeValueAsString(new AddressDto("Budapest", "Andrássy u. 2."));

        server.expect(requestTo(startsWith("http://localhost:8080/api/addresses")))
                .andExpect(queryParam("name", "John%20Doe"))
                .andRespond(withSuccess(json, MediaType.APPLICATION_JSON));

        AddressDto addressDto = addressesGateway.findAddressByName("John Doe");

        assertEquals("Budapest", addressDto.getCity());
        assertEquals("Andrássy u. 2.", addressDto.getAddress());
    }
}
```

## WireMock

```xml
<dependency>
	<groupId>com.github.tomakehurst</groupId>
	<artifactId>wiremock-jre8</artifactId>
	<version>${wiremock.version}</version>
	<scope>test</scope>
</dependency>
```

```java
import com.github.tomakehurst.wiremock.WireMockServer;
import com.github.tomakehurst.wiremock.client.WireMock;

import static com.github.tomakehurst.wiremock.client.WireMock.*;
import static com.github.tomakehurst.wiremock.core.WireMockConfiguration.wireMockConfig;
```


```java
public class AddressesGatewayIT {

    static String host = "127.0.0.1";

    static int port;

    static WireMockServer wireMockServer;

    @BeforeAll
    static void startServer() {
        port = SocketUtils.findAvailableTcpPort();
        wireMockServer = new WireMockServer(wireMockConfig().port(port));
        WireMock.configureFor(host, port);
        wireMockServer.start();
    }

    @AfterAll
    static void stopServer() {
        wireMockServer.stop();
    }

    @BeforeEach
    void resetServer() {
        WireMock.reset();
    }
}
```

```java
@Test
void testFindAddressByName() {
  String resource = "/api/addresses";

  stubFor(get(urlPathEqualTo("/api/addresses"))
    .willReturn(aResponse()
    .withHeader("Content-Type", "application/json")
    .withBody("{\"city\": \"Budapest\", \"address\": \"Andrássy u. 2.\"}")));

  String url = String.format("http://%s:%d/api/addresses?name={name}", host, port);
  AddressesGateway gateway = new AddressesGateway(new RestTemplateBuilder(), url);
  Address address = gateway.findAddressByName("John Doe");

  verify(getRequestedFor(urlPathEqualTo(resource))
    .withQueryParam("name", equalTo("John Doe")));
    
  assertThat(address.getCity()).isEqualTo("Budapest");
  assertThat(address.getAddress()).isEqualTo("Andrássy u. 2.");
}
```

## JMS

```yaml
version: '3'

services:
  eventstore-mq:
    image: training360/eventstore-mq
    restart: always
    ports:
      - "8161:8161"
      - "61616:61616"
  eventstore:
    image: training360/eventstore
    restart: always
    depends_on:
      - eventstore-mq
    ports:
      - "8082:8080"
    environment:
      SPRING_ARTEMIS_HOST: 'eventstore-mq'
    entrypoint: ["./wait-for-it.sh", "-t", "120", "eventstore-mq:61616", "--", 
               "java", "org.springframework.boot.loader.JarLauncher"]
```

```
docker-compose up
```

* Elérhető a `http://localhost:8161` címen.
* Alapértelmezett felhasználónév/jelszó: `admin` / `admin`

* Alkalmazás: `localhost:8082`

## JMS küldés

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-artemis</artifactId>
</dependency>
```

* `JmsConfig`

```java
@Bean
public MessageConverter messageConverter(ObjectMapper objectMapper){
  MappingJackson2MessageConverter converter =
  new MappingJackson2MessageConverter();
  converter.setTypeIdPropertyName("_typeId");
converter.setTypeIdMappings(
  Map.of("CreateEventCommand", EmployeeHasCreatedEvent.class));
  return converter;
}
```

```java
EmployeesEvent
  message
```


```java
public class EventStoreGateway {

    private JmsTemplate jmsTemplate;

    public EventStoreGateway(JmsTemplate jmsTemplate) {
        this.jmsTemplate = jmsTemplate;
    }

    public void sendEvent(String message) {
        log.debug("Send event to eventstore");
        jmsTemplate.convertAndSend("eventsQueue", new EmployeesEvent(message));
    }
}
```

## Fogadás

* `internalQueue`

```java
@JmsListener(destination = "eventsQueue")
public void processMessage(EmployeesEvent event) {
    log.info("" + event.getMessage());
}
```

## Actuator

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

`management.endpoints.web.exposure.include = *`

```properties
management.endpoint.shutdown.enabled = true
```


```properties
management.endpoint.health.show-details = always
```

* `/health`
* Heap dump: `/heapdump` (bináris állomány)
* Thread dump: `/threaddump`
* Beans: `/beans`
* Conditions: `/conditions`
    * Autoconfiguration feltételek teljesültek-e vagy sem - ettől függ, <br /> milyen beanek kerültek létrehozásra
* HTTP mappings: `/mappings`
    * HTTP mappings és a hozzá tartozó kiszolgáló metódusok
* Configuration properties: `/configprops`
* `/env` végpont - property source-ok alapján felsorolva

* `/flyway` - Flyway

## HttpTrace

* Ha van `HttpTraceRepository` a classpath-on
* Fejlesztői környezetben: `InMemoryHttpTraceRepository`
* Éles környezetben: Zipkin vagy Spring Cloud Sleuth
* Megjelenik a `/httptrace` endpoint

## Info

* `info` prefixszel megadott property-k belekerülnek

```properties
info.appname = employees
```

```xml
<plugin>
  <groupId>pl.project13.maven</groupId>
  <artifactId>git-commit-id-plugin</artifactId>
</plugin>
```

`mvn package`

## Naplózás

* `/loggers`
* `/logfile`

```plaintext
### Get logger
GET http://localhost:8080/actuator/loggers/training.employees

### Set logger
POST http://localhost:8080/actuator/loggers/training.employees
Content-Type: application/json

{
  "configuredLevel": "INFO"
}
```

## Metrics

* `/metrics` végponton

```java
Counter.builder(EMPLOYEES_CREATED_COUNTER_NAME)
        .baseUnit("employees")
        .description("Number of created employees")
        .register(meterRegistry);

meterRegistry.counter(EMPLOYEES_CREATED_COUNTER_NAME).increment();
```

* A `/metrics/employees.created` címen elérhető

## Metrics Graphite monitoring eszközzel

(Küld)

```shell
docker run
  -d
  --name graphite
  --restart=always
  -p 80:80 -p 2003-2004:2003-2004 -p 2023-2024:2023-2024
  -p 8125:8125/udp -p 8126:8126
  graphiteapp/graphite-statsd
```

Felhasználó/jelszó: `root`/`root`

```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-graphite</artifactId>
  <version>${micrometer-registry-graphite.version}</version>
</dependency>
```

```properties
management.metrics.export.graphite.step = 10s
```

# Metrics Prometheus monitoring eszközzel

(Lekérdez)

```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
  <version>${micrometer-registry-prometheus.version}</version>
</dependency>
```

* `/actuator/prometheus` endpoint

* yml konfiguráció, `prometheus.yml`

```yaml
scrape_configs:
  - job_name: 'spring'
    metrics_path: '/actuator/prometheus'
    scrape_interval: 20s
    static_configs:
      - targets: ['host.docker.internal:8080']
```

Tegyük fel, hogy a `prometheus.yml` a `D:\data\prometheus` könyvtárban van

```shell
docker run -p 9090:9090 -v D:\data\prometheus:/etc/prometheus
  --name prom prom/prometheus
```

## Audit events

```java
applicationEventPublisher.publishEvent(
  new AuditApplicationEvent("anonymous",
    "employee_has_been_created",
      Map.of("name", command.getName())));
```

* `InMemoryAuditEventRepository`
* Megjelenik az `/auditevents` endpoint

## Continuous Delivery Jenkins Pipeline-nal


```
FROM jenkins/jenkins:lts-jdk11
ENV JAVA_OPTS -Djenkins.install.runSetupWizard=false

RUN jenkins-plugin-cli --plugins "git workflow-aggregator pipeline-stage-view docker-plugin docker-workflow locale blueocean"
```

```shell
git update-index --chmod=+x mvnw
```

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-failsafe-plugin</artifactId>
	<version>2.22.2</version>
	<executions>
		<execution>
			<goals>
				<goal>integration-test</goal>
				<goal>verify</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```

```groovy
pipeline {
   agent any

   stages {
      stage('package') {
         steps {
            git 'http://gitlab.training360.com/trainers/employees-ci'

            sh "./mvnw clean package"
         }
      }
      stage('test') {
         steps {
            sh "./mvnw verify"
         }
      }
   }
}
```

# MapStruct

modelmapper -> comment

<mapstruct.version>1.4.2.Final</mapstruct.version>

<dependency>
			<groupId>org.mapstruct</groupId>
			<artifactId>mapstruct</artifactId>
			<version>${mapstruct.version}</version>
		</dependency>

<plugin>
   	<groupId>org.apache.maven.plugins</groupId>
   	<artifactId>maven-compiler-plugin</artifactId>
   	<version>3.8.1</version>
   	<configuration>
   		<annotationProcessorPaths>
   			<path>
   				<groupId>org.projectlombok</groupId>
   				<artifactId>lombok</artifactId>
   				<version>${lombok.version}</version>
   			</path>
   			<path>
   				<groupId>org.mapstruct</groupId>
   				<artifactId>mapstruct-processor</artifactId>
   				<version>${mapstruct.version}</version>
   			</path>
   		</annotationProcessorPaths>
   	</configuration>

</plugin>

EmployeesApplication -> kivenni

package employees;

import org.mapstruct.Mapper;

import java.util.List;

@Mapper(componentModel = "spring")
public interface EmployeeMapper {

    List<EmployeeDto> toDto(List<Employee> employees);

    EmployeeDto toDto(Employee employee);
}
