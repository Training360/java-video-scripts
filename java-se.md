
2020-10-13

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
