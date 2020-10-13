
2020-10-13

# installjdk (nem került felvételre!!!)

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

# cmdarguments

`C:\trainer\java-se` könyvtár, `cmdarguments` csomag.

`CommandLineArguments` `main()` metódus

`args.length`, `args[0]`

IDEA-ban, majd parancssorból

java-se-module03-lesson01p5.mp4

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
