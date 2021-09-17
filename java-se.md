# installjdk

* Telepítés
* Környezeti változók

java-se-module01-lesson01p5-lab.mp4

# introide

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>15</maven.compiler.source>
    <maven.compiler.target>15</maven.compiler.target>
</properties>
```

java-se-module01-lesson04-lab.mp4

# introgit

* Projekt létrehozás: `C:\training\introgit`

```java
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>15</maven.compiler.source>
    <maven.compiler.target>15</maven.compiler.target>
</properties>
```
* `.gitignore`
* Új osztály `HelloWorld`
* Git/Init
* Add
* Commit
* Módosítás
* Diff
* Commit
* Módosítás
* Revert

java-se-module01-lesson05-lab.mp4

# introgithub

`introgit` projekten

* Projekt létrehozása
* Settings - Git auth
* Remote hozzáadása
* Push
* Változtatás és Git commit and push

java-se-module01-lesson05p5-lab.mp4

# cmdarguments

`C:\trainer\java-se` könyvtár, `cmdarguments` csomag.

`CommandLineArguments` `main()` metódus

`args.length`, `args[0]`

IDEA-ban, majd parancssorból

java-se-module03-lesson01p5.mp4

# references

```java
public static void main(String[] args) {
    int yearOfBirth = 1980;

    String name = "John Doe";

    Trainer john = new Trainer("John Doe", 1980);

    Trainer trainer = john;

    System.out.println(trainer == john);

    Trainer john2 = new Trainer("John Doe", 1980);

    System.out.println(trainer == john2);
}
```

* `this` kulcsszó használata

java-se-module04-lesson05-lab.mp4

# junit5

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.5.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

* `@Test` annotáció
* Régi kód
* Új: `import`, `public` nem kell, `assertEquals()`
* Hamcrest deprecated

```java
@Test
public void testCreate() {
    // Given
    Trainer trainer = new Trainer("John Doe");

    // When
    String name = trainer.getName();

    // Then
    assertThat(name, equalTo("John Doe"));
}
```

java-se-module06-lesson01p5.mp4

# interface

`Trainer` osztály, `Human` interfész

Deklarálok egy `Trainer` majd egy `Human` típusú változót, majd egy lépésben

Új `HasName` interfész, `public String getName()` metódus

`Trainer` osztály ezt implementálja

`Course` osztály is implementálja ezt

java-se-module12-lesson02-lab.mp4

# interfacerules

* `Trainer`, `HasName`, `getName()` metódus `CanWork`, `doWork()` metódus
* Interfész nem példányosítható

```java
public interface EmployeeType {
  int FULL_TIME = 0;
  
  int PART_TIME = 1;
}
```

java-se-module12-lesson02p3-lab.mp4

# interfacedependencyinversion

Kiindulni, hogy van egy `Employee`, `int salary` `return salary * 1,25`.

```java
public interface BonusCalculator {
  int calculateBonus(int salary);
}
```

```java
public class Employee {
  
  private int salary;
  
  private BonusCalculator calculator;

  public Employee(int salary, BonusCalculator calculator) {
    this.salary = salary;
    this.calculator = calculator;
  }
  
  public int getBonus() {
    return calculator.getBonus(salary);
  }    
  
}
```

```java
public class PercentBonusCalculator implements BonusCalculator {
  public int calculateBonus(int salary) {
    return salary * 1.25;
  }
}
```

```java
public class FixBonusCalculator implements BonusCalculator {
  public int calculateBonus(int salary) {
    return salary + 10_000;
  }
}
```

java-se-module12-lesson02p4-lab.mp4

# exceptions

```java
try {
  Scanner scanner = new Scanner(NumbersReader.class.getResourceAsStream("numbers.csv"));
  while (scanner.hasNextLine()) {
    String line = scanner.nextLine();
    int number = Integer.parseInt(line);
  }
}
catch (NumberFormatException e) {
  // ...
}
catch (Exception e) {
  // ...
}
finally {
  scanner.close();
}
```

Kivételt magát is tovább lehet dobni, vagy becsomagolni

java-se-module14-lesson01-lab.mp4


# aslist

```java
List<String> names = new ArrayList<>();
names.add("John"); // Kíírás hozzáadásonként

List<String> names = Arrays.asList("John", "Jack", "Jane"); // elem hozzáadása

List<String> names = new ArrayList(Arrays.asList("John", "Jack", "Jane"));
```
