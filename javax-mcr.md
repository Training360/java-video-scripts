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
