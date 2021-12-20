```xml
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-framework-bom</artifactId>
                <version>5.0.1.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
        </dependency>
```

```java
public class EmployeesDao {

    private JdbcTemplate jdbcTemplate;

    public EmployeesDao(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }

    public long createEmployee(String name) {
        KeyHolder holder = new GeneratedKeyHolder();

        jdbcTemplate.update(con -> {
            PreparedStatement ps = con.prepareStatement("insert into employees(emp_name) values (?)",
                    Statement.RETURN_GENERATED_KEYS);
            ps.setString(1, name);
            return ps;
        }, holder);

        return holder.getKey().longValue();
    }

    public List<String> listEmployeeNames() {
        return jdbcTemplate.query("select emp_name from employees order by emp_name",
                (rs, rowNum) -> rs.getString("emp_name"));
    }

    public String findEmployeeNameById(long id) {
        return jdbcTemplate.queryForObject("select emp_name from employees where id = ?",
                new Object[]{id}, (rs, rowNum) -> rs.getString("emp_name"));
    }
}
```

```java
public class EmployeesDaoTest {

    private EmployeesDao employeesDao;

    @Before
    public void init() {
        MysqlDataSource dataSource = new MysqlDataSource();
        dataSource.setUrl("jdbc:mysql://localhost:3306/employees?useUnicode=true");
        dataSource.setUser("employees");
        dataSource.setPassword("employees");

        Flyway flyway = new Flyway();
        flyway.setDataSource(dataSource);

        flyway.clean();
        flyway.migrate();

        employeesDao = new EmployeesDao(dataSource);
    }

    @Test
    public void testCreateThanList() {
        employeesDao.createEmployee("John Doe");
        List<String> employees = employeesDao.listEmployeeNames();

        assertEquals(Arrays.asList("John Doe"), employees);
    }

    @Test
    public void testThanFind() {
        long id = employeesDao.createEmployee("John Doe");
        System.out.println(id);
        String name = employeesDao.findEmployeeNameById(id);
        assertEquals("John Doe", name);
    }
}
```

# JPA

```xml
<dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.41</version>
        </dependency>

<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-entitymanager</artifactId>
    <version>5.3.6.Final</version>
</dependency>

<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.0</version>
</dependency>
```

* persistence unit `src/main/resources/META-INF`

```java
@Entity
@Table(name = "employees")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "emp_name")
    private String name;

    public Employee() {        
    }

    public Employee(String name) {
        this.name = name;
    }

    // Getter és setter metódusok
}
```

```java
EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("pu");
EntityManager entityManager = entityManagerFactory.createEntityManager();
entityManager.getTransaction().begin();
Employee employee = new Employee("John Doe");
entityManager.persist(employee);
entityManager.getTransaction().commit();
```

# További műveletek

```java
List<Employee> employees = entityManager
  .createQuery("select e from Employee e order by e.name", 
      Employee.class)
    .getResultList();
```

```java
Employee employee = entityManager.find(Employee.class, 1);
employee.setName("Jack Doe");
```

```java
Employee employee = entityManager.find(Employee.class, 1);
entityManager.remove(employee);
```

# Tesztadatok előkészítése

JDBC flyway
 - lefuttatom egyszer: létrehozza a táblát, van benne adat
 - lefuttatom még egyszer - ugyanúgy egy rekord van benne, előzőt kitörli
 - clean törlése
 - használjuk teszt adat gyártására
 - átírjuk - elszáll
 - új insert, éles adatbázisba is bekerül
 - nem használjuk teszt adat gyártására, csak fejlesztés során új táblák létrehozására
- testFindEmployeeNameById 
