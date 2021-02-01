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

