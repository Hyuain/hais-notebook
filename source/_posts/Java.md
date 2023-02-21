---
title: Java
date: 2023-02-09 17:14:22
categories:
  - [全栈]
---

.

<!-- more -->

# Concepts

## Object Oriented

- **Encapsulation**: a class contains data and behaviors, and can hide some
- **Abstraction**
  - Abstract away implementation details
  - Isolate the impact of changes made to the code
- **Inheritance**: It weakens encapsulation.
- **Polymorphism**: Any object instantiated by any child class can be handled in the same way
- **Coupling and Cohesion**: *Loose coupled, highly cohesive*
  - **Coupling** (dependency): The degree to which one class knows about another class.
  - **Cohesion**: The degree to which a class has a single, well-focused purpose.

- **Interface/Implementation Paradigm**
  - An object should **reveal only the interfaces** that other objects must have to interact with it.
  - Clients of the class will not be affected by implementation change.


## UML Class Diagrams

[UML Class Diagram Tutorial Video](https://www.youtube.com/watch?v=UI6lqHOVHic)

[UML Class Diagram Tutorial](https://www.visual-paradigm.com/guide/uml-unified-modeling-language/uml-class-diagram-tutorial/;WWWSESSIONID=011FBA1C6F02E5074E7D3E435507A547.www1)

### Attributes

- Data containing values that describe each instance of that class
- Also called fields, variables, properties.

### Methods

- Specify behavioral feature of a class.
- Also called operations or functions.

### Visibility

```text
- private
+ public
# protected
~ package/default
```

### Relationship

![Class Relationship](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/ClassRelationship.jpg)

![Inheritance](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CDInheritance.jpg)

![Association](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CDAssociation.jpg)

![Aggregation](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CDAggregation.jpg)

![Composition](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CDComposition.jpg)

### Example

![Class Diagram Example](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CDExample1.jpg)

## SOLID Principle

- SRP: Single Responsibility Principle
- OCP: Open/Close Principle
- LSP: Liskov Substitution Principle
- IPS: Interface Segregation Principle
- DIP: Dependency Inversion Principle

### SRP: Single Responsibility Principle

```text
Animal
- perfiomWalk()
- performSwim()
- performSleep()
```

### OCP: Open/Close Principle

- Software entities (classes, modules, functions, etc.) should be:
  - Open for extension
  - Close for modification
- Be able to extend a class's behavior without modifying it.

### LSP: Liskov Substitution Principle

If we substitute a super class object reference with an object of *any* of its subclasses, the program should not break. 

If a superclass can do something, a subclass **MUST** also be able to do it.

### IPS: Interface Segregation Principle

No code should be forced to depend on methods it does not use.

Splits large interfaces into smaller and more specific ones.

```java
// Large interface
public interface Zookeeper {
  void washAnimal();
  void feedAnimal();
  void petAnimal();
}

// Split into ...
public interface AnimalCleaner {
  void washAnimal();
}
public interface AnimalFeeder {
  void feedAnimal();
}
public interface AnimalPetter {
  void petAnimal();
}

class AnimalCarer implements AnimalCleaner, AnimalFeeder {
	//...
}
class AnimalLover implements AnimalPetter {
  //...
}
```

### DIP: Dependency Inversion Principle

Depend upon abstractions. Do not depend upon concrete classes.

## Convention-Over-Configuration Principle

The convention represents the most-used way to configure the app for a specific purpose.

Only need to change those places where your app needs more particular configuration.

Write less code for configuration.

# Startup

## Extension Pack for Java for VSCode

1. Install the `Extension Pack for Java`
2. Refresh VSCode: Press `Ctrl+Shift+P` and input `Reload Window`
3. Clean the Java workspace when there are problems: Press `Ctrl+Shift+P` and input `Java: Clean Java Language Server Workspace`

## Creating and Running Java programs

Create Java source file `TestCircle.java`:

```java
// TestCircle.java
public class TestCircle {
  public static void main(String args[]) {
    System.out.println("Hello");
  }
}
```

Compile the Java source file into bytecode:

```bash
javac TestCircle.java
```

Execute the Java program:

```java
java TestCircle
```

To generate Setter and Getter methods in VSCode, in a Java source file, right-click and select `Source Action...`. And then select `Generate Getters and Setters`. The way to generate constructors is similar.

# OO Design in Java

## Encapsulation

- Access Modifier
  - The keyword `private` signifies that a method or variable can be accessed only within the declaring object.
  - All/most attributes should be declared as private.
-  Getters and Setters

## Inheritance

- **Is-a Relationship**: A subclass can do anything that the superclass can do.
  - **Generalization**: A subclass is a type of superclass (a dog **is-a** mammal)

### Method Overriding

```java
public class Animal {
  public void sound() {
    //...
  }
}

public class Horse extends Animal {
  // Not mandatory but considered as best practice for coding
  // Because it will let comiler to check wether Animal has sound or not
  @Override
  public void sound() {}
}
```

### Interface and Abstract Class

- An **interface** can provide **NO implementation** at all.
  - A class can inherit from only one parent class, but it can implement many interfaces.
- An **abstract class** can provide some implementation.

```java
interface Nameable {
  String getName();
  void setName(String aName);
}

abstract class Mammal {
  public void generateHeat() {
    //...
  }
  public abstract void makeNoise();
}
  
class Dog extends Mammal implements Nameable {
  String name;
  
  public void makeNoise() {
    //...
  }
  public void setName (String aName) {
    name = aName;
  }
  public String getName() {
    return name;
  }
}

Dog D = new Dog();
Mammal M = D;
Nameable N = D;
```

## Composition

- **Has-a Relationship**: A dog **has a** head.

```java
class Mammal {
  public void eat() {
    //...
  }
}

public Walkable {
  public void walk() {
    //...
  }
}

class Dog {
  Mammal m = new Mammal();
  Walkable w = new Walkable();
  
  public void performWalk() {
    w.walk();
  }
  public void performEat() {
    m.eat();
  }
}
```

## Inverting of Control (IoC)

- **DIP**: high-level modules should **not** depend on low-level modules.
- **Both** should depend on abstractions.

![High-level module depends on low level module](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/java-before-ioc.jpg)

![Inverting of Control](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/java-ioc.jpg)

Create 2 interfaces: to fetch the data and export the data.

- All low-level modules must implements these interfaces
- High-level module (Balance Sheet Module) needs to rely on interfaces

```java
public interface IFetchData {
  List<Object[]> fetchData();
}
public interface IExportData {
  File exportData(List<Object[]> listData);
}
```

```java
public class BalanceSheet {
  private IExportData exportDataObj = null;
  private IFetchData fetchDataObj = null;
  
  public Object generateBalanceSheet() {
    List<Object[]> dataList = fetchDataObj.fetchData();
    return exportDataObj.exportData(dataList);
  }
}
```

### Dependency Injection (DI)

- DI is one way to implement IoC.
- Decouples your class's construction from the construction of its dependencies.
  - Invert the object creation process from your module to other code or entity.
  - Inject the object from outside.

#### Constructor Injection

```java
public class BalanceSheet {
  private IExportData exportDataObj;
  private IFetchData fetchDataObj;
  
  BalanceSheet(IFetchData fetchData, IExportData exportData) {
    this.fetchDataObj = fetchData;
    this.exportDataObj = exportData;
  }
}
```

#### Setter Injection

```java
public class BalanceSheet {
  private IExportData exportDataObj;
  private IFetchData fetchDataObj;
  
  public void setExportDataObj(IExportData exportDataObj) {
    this.exportDataObj = exportDataObj;
    this.fetchDataObj = fetchDataObj;
  }
}
```

### IoC Containers

- A container takes care of creating, configuring, and managing objects.
- You just need to do configuration, and the container will take care of object instantiation and dependency management with ease.
- Java IoC Containers: Spring, Google Guice and Dagger.

### Factory Pattern

```java
public class BalanceSheet {
  private IFetchData fetchDataObj;
  
  public void configureFetchData(String type) {
    this.fetchDataObj = FetchDataFactory.getFetchData(type);
  }
}

public class FetchDataFactory {
  public static IFetchData getFetchData(String type) {
    IFetchData fetchData = null;
    if ("FROM_DB".equalsIgnoreCase(type)) {
      fetchData = new FetchDatabase();
    }
    else if ("FROM_WS".equalsIgnoreCase(type)) {
      fetchData = new FetchWebService();
    }
    else {
      return null;
    }
    return fetchData;
  }
}
```



# Maven

Maven is a tool for building and managing your Java-based projects.

## Project Structure

```text
|-- pom.xml
`-- src
  |--main
  |  `-- java
  |    `-- com
  |      `-- example
  |        `-- App.java
  `-- test
    `-- java
      `-- com
        `-- example
          `-- AppTest.java
```

- **src/main/java**: The source code.
- **src/main/resources**: The resources such as configuration files and property files.
- **src/test/resources**: The test resources such as configuration files and property files.
- **pom.xml:** **Project Object Model**. The project's information and dependencies.
  - **groupId**: identifies the group or organization that the dependency belongs to.
  - **artifactId**: the id of the artifact within the group.
  - **version**: the version of the dependency that the project requires.
  - **scope**: the scope of the dependency (whether it is required for compilation, test, runtime, etc.)
- **target**: The compiled code and other files generated during the build process.

## Create a Maven Project

### Use VSCode Extension

Step 1:

In VSCode, press `Ctrl+Shift+P`. Search for Java, Select `Java: Create Java Project`.

Alternatively, you may click `+` under the **Java Project Panel**.

Step 2: Choose `Maven`.

Step 3: Select `maven-archetype-quickstart`

### Use Command

```bash
mvn archetype:generate -DgroupId=com.example \
-DartifactId=Demo2 \
-DarchetkypeArtifactId=maven-archetype-quickstart \
-DinteractiveMode=false
```

## Maven Lifecycle

3 build-in build lifecycles:

- **Default**: handles project deployment.
- **Clean**: handles project cleaning.
- **Site**: handles the creation of project's web site.

These are some phases in the default lifecycle:

- **validate**
- **compile**
- **test**: unit test
- **package**
- **verify**: integration test
- **install**: install the package into local repository
- **deploy**

## Goals

- A specific task (finer than a build phase) which contributes to the building and managing of a project.
- Each goal is bound to one or more phases.
  - Corresponds to a specific action that needs to be performed
  - E.g. compiling code, running tests, and packaging the code.
- One or more plugins can be bounded to the goal.

## Plugins

- Perform tasks in specific goal(s).
  - E.g. automate common build tasks, such as compiling code, running tests, and packaging the code.
- Can be bound to one or more goals.

## Commands

![Maven Phases, goals and Plugins](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/maven-phase-goal-plugin.jpg)

A Maven command may be written in 2 forms:

```bash
mvn [phase]
mvn [plugin]:[goal]
```

E.g.:

```bash
mvn compile            # execute default lifecycle till compile phase
mvn compiler:compile   # only excute compiler plugin to comile

mvn test
mvn surefire:test      # only excute surefire plugin to test

mvn exec:java
```

## JAR (Java Archive) File

A package file format used to aggregate many Java class files and associated metadata and resources into one file for distribution.

# Spring Boot

## Simple Demo With Rest API

### Spring Initializr

- [Spring Initializr](https://start.spring.io/) helps you initialize spring boot project.

- Choose Spring Web, Spring Data JPA and PostgreSQL Driver.
- Use IntelliJ to open the project. 

- Since we have not installed database, comment out `spring-boot-starter-data-jpa` in `pom.xml`, and then run the project.

- Open browser and go to localhost:8080

### Simple API

In `DemoApplication.java`

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@SpringBootApplication
@RestController
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

	@GetMapping("/xx")
	public List<String> hello() {
		return List.of("Hello World", "123");
	}

}
```

### Student Class

- Right click `com.example.demo` folder, click new package to create a `com.example.demo.student` package.

- Create a `Student` class inside the package.

```java
package com.example.demo.student;

import java.time.LocalDate;

public class Student {
    private Long id;
    private String name;
    private String email;
    private LocalDate dob;
    private Integer age;

  	// Use Ctrl+Insert to generate construtors, getters, setters, toString quickly
    public Student() {
    }

    public Student(Long id, String name, String email, LocalDate dob, Integer age) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.dob = dob;
        this.age = age;
    }

    public Student(String name, String email, LocalDate dob, Integer age) {
        this.name = name;
        this.email = email;
        this.dob = dob;
        this.age = age;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public LocalDate getDob() {
        return dob;
    }

    public void setDob(LocalDate dob) {
        this.dob = dob;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", dob=" + dob +
                ", age=" + age +
                '}';
    }
}
```

And then go to `DemoApplication`

```java
	@GetMapping("/xx")
	public List<Student> hello() {
		return List.of(
				new Student(
						1L,
						"Mariam",
						"Mariam.jamal@gmail.com",
						LocalDate.of(2000, Month.JANUARY, 5),
						21
				)
		);
	}
```

### API Layer

Create `StudentController` class inside student package, and remove relative code from `DemoApplication`

```java
package com.example.demo.student;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDate;
import java.time.Month;
import java.util.List;

@RestController
@RequestMapping(path = "api/v1/student")
public class StudentController {
    @GetMapping
    public List<Student> getStudents() {
        return List.of(
                new Student(
                        1L,
                        "Mariam",
                        "Mariam.jamal@gmail.com",
                        LocalDate.of(2000, Month.JANUARY, 5),
                        21
                )
        );
    }
}

```

### Business Layer and Dependency Injection

Create a `StudentService` class inside student package, and modify `StudentController` like below:

```java
// StudentService.java
package com.example.demo.student;

import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;

import java.time.LocalDate;
import java.time.Month;
import java.util.List;

// @Component or @Service tells the class should be instantiated (It is a bean)
@Service
public class StudentService {
    public List<Student> getStudents() {
        return List.of(
                new Student(
                        1L,
                        "Mariam",
                        "Mariam.jamal@gmail.com",
                        LocalDate.of(2000, Month.JANUARY, 5),
                        21
                )
        );
    }
}
```

```java
// StudentController.java
package com.example.demo.student;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping(path = "api/v1/student")
public class StudentController {

    private final StudentService studentService;

    // @Autowired annotation tells inject StudentService automatically
    @Autowired
    public StudentController(StudentService studentService) {
        this.studentService = studentService;
    }

    @GetMapping
    public List<Student> getStudents() {
        return studentService.getStudents();
    }
}
```

Use dependency injection to inject `StudentService` into `StudentController`.

### Create Database and Properties File

Modify the `application.properties` file (Sometimes `username` and `password` are also needed.):

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/student
spring.datasource.username=
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.format_sql=true
```

Install PostgreSQL. (Sometimes you need to turn on postgresql service in windows task manager.)

Enter psql using `psql` command or SQL Shell (psql). Use `\l` to see existed databases.

Then, create student database:

```sql
CREATE DATABASE student;
```

Use `\du` to see list of roles.

Then, grant privileges:

```sql
GRANT ALL PRIVILEGES ON "student" TO [username];
```

Then, connect to student database:

```bash
\c student
```

Use `\d` to see tables in student database.

Uncomment `spring-boot-starter-data-jpa` in `pom.xml`

### JPA

Modify `Student` class:

```java
@Entity
@Table
public class Student {
    @Id
    @SequenceGenerator(
            name = "student_sequence",
            sequenceName = "student_sequence",
            allocationSize = 1
    )
    @GeneratedValue(
            strategy = GenerationType.SEQUENCE,
            generator = "student_sequence"
    )
    private Long id;
  	//...
}
```

### JPA Repository

Create a `StudentRepository` interface inside student package:

```java
package com.example.demo.student;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

// JpaRepository<Type, ID Type>
@Repository
public interface StudentRepository extends JpaRepository<Student, Long> {

}
```

Inject into `StudentService` class:

```java
@Service
public class StudentService {

    private final StudentRepository studentRepository;

    @Autowired
    public StudentService(StudentRepository studentRepository) {
        this.studentRepository = studentRepository;
    }
    public List<Student> getStudents() {
        return studentRepository.findAll();
    }
}
```

### Saving Student

Create a `StudentConfig` class inside student package:

```java
package com.example.demo.student;

import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.LocalDate;
import java.time.Month;
import java.util.List;

@Configuration
public class StudentConfig {

    @Bean
    CommandLineRunner commandLineRunner(StudentRepository repository) {
        return args -> {
            Student mariam = new Student(
                    1L,
                    "Mariam",
                    "Mariam.jamal@gmail.com",
                    LocalDate.of(2000, Month.JANUARY, 5),
                    21
            );
            Student alex = new Student(
                    2L,
                    "Alex",
                    "Alex@gmail.com",
                    LocalDate.of(2004, Month.JANUARY, 5),
                    21
            );
            repository.saveAll(
                    List.of(mariam, alex)
            );
        };
    }
}

```

Restart the server, and 2 students above will be saved in student database, and can be seen using `\d`.

### Transient

Add `@Transient` notation to `age`, since age can be calculated.

```java
@Entity
@Table
public class Student {
    @Transient
    private Integer age;
    //...
    public Integer getAge() {
        return Period.between(this.dob, LocalDate.now()).getYears();
    }

}
```

 Restart the server, and age will no longer be a column in the table.

### Add Student

Use `@PostMapping` in `StudentController`.

`@RequestBody` can map request body to class.

```java
@PostMapping
public void registerNewStudent(@RequestBody Student student) {
    studentService.addNewStudent(student);
}
```

Click the network icon to generate http-client for testing:

```http
###
POST http://localhost:8080/api/v1/student
Content-Type: application/json

{
  "name": "Bilal",
  "email": "bilal.ahmed@gmail.com",
  "dob": "1995-12-17"
}
```

Complete `addNewStudent` method in `StudentService`:

```java
  public void addNewStudent(Student student) {
      Optional<Student> studentOptional = studentRepository.findStudentByEmail(student.getEmail());
      if (studentOptional.isPresent()) {
          throw new IllegalStateException("email taken");
      }
      studentRepository.save(student);
      System.out.println(student);
  }
```

And `findStudentByEmail` method in `StudentRepository`:

```java
// @Query can be omitted
@Query("SELECT s FROM Student s WHERE s.email = ?1")
Optional<Student> findStudentByEmail(String email);
```

Modify `application.properties` to send error message to front end:

```properties
server.error.include-message=always
```

### Delete Student

Use `@DeleteMapping` in `StudentController`:

```java
@DeleteMapping(path = "{studentId}")
public void deleteStudent(@PathVariable("studentId") Long studentId) {
    studentService.deleteStudent(studentId);
}
```

Complete `StudentService`:

```java
public void deleteStudent(Long studentId) {
    boolean exists = studentRepository.existsById(studentId);
    if (!exists) {
        throw new IllegalStateException("student with id " + studentId + "does not exists");
    }
    studentRepository.deleteById(studentId);
}
```

### Update Student

Use `@PutMapping` in `StudentController`:

```java
@PutMapping(path = "{studentId}")
public void updateStudent(
        @PathVariable("studentId") Long studentId,
        @RequestParam(required = false) String name,
        @RequestParam(required = false) String email
) {
    studentService.updateStudent(studentId, name, email);
}
```

Complete `updateStudent` in `StudentService`:

```java
@Transactional
public void updateStudent(Long studentId, String name, String email) {
    Student student = studentRepository.findById(studentId)
            .orElseThrow(() -> new IllegalStateException(
                    "student with id " + studentId + " dose not exist"
            ));
    if (name != null && name.length() > 0 && !Objects.equals(student.getName(), name)) {
        student.setName(name);
    }
    if (email != null && email.length() > 0 && !Objects.equals(student.getEmail(), email)) {
        Optional<Student> studentOptional = studentRepository.findStudentByEmail(email);
        if (studentOptional.isPresent()) {
            throw new IllegalStateException("email taken");
        }
        student.setEmail(name);
    }
}
```

### Packaging

Stop the servers, open Maven panel, and run `clean`. It will delete the `target` folder.

Then run `install`, get the `target folder`. There is a `demo-0.0.1-SNAPSHOT.jar` file in this folder.

Open the terminal:

```bash
cd target
java -jar demo-0.0.1-SNAPSHOT.jar
```

Now the application is running.

And we can change the port by adding `--server.port=8081` at the end.

## Generate a Spring Boot Project

[Spring Initializr](https://start.spring.io/) configured into your Maven project

- The Spring app main class
- The Spring Boot POM parent
- The dependencies
- The Spring Boot Maven plugin
- The properties file

## Dependency Starter

A dependency starter is a group of dependencies you add to configure your app for a specific purpose.

![Dependency Starter](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/dependency-starter.jpg)

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## The Spring Ecosystem

- Spring Core
- Spring Model-View-Controller (MVC): Web application development.
- Spring Data Access: Connect to SQL databases.
- Spring Testing

## @SpringBootApplication

Enables

- **@EnableAutoConfiguration**: Automatically configure your Spring application based on the jar dependencies that you have added. (e.g., Configure database connection based on specified dependency)
- **@ComponentScan**: Enable scan **@Component**.
- @**Configuration**: Allow register extra beens in the context or import additional configuration classes.

## Spring Context

The place in the app's memory where we add the object instances we want Spring to manage.

### @Bean

A bean is an object that is instantiated, assembled, and managed by a Spring IoC container.

![Spring Context](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/SpringContext.jpg)

### @Configuration

Configuration class is a special class to instruct Spring to do specific actions.

```java
@Configuration
public class ProjectConfig {
  @Bean
  Parrot parrot() {
    var p = new Parrot();
    p.setName("Koko");
    return p;
  }

}
```

### @Component

Mark it as a component. Spring creates an instance of the class and adds that instance to its context.

## Dependency Injection (DI) in Spring

### @Autowired

Defines the object that needs the dependency.

If there is a SMSService which implements MessageService:

```java
@Component
public class SMSService implements MessageService {
}
```

#### Field-Based Injection

```java
@Component
public class Client {
  @Autowired
  private MessageService messageService;
}
```

#### Constructor-Based Injection

```java
@Component
public class Client {
  private final MessageService messageService;
  
  @Autowired
  public Client(MessageService messageService) {
    this.messageService = messageService;
  }
}
```

#### Setter-Based Injection

```java
@Component
public class Client {
  private MessageService messageService;
  
  @Autowired(required = false)
  public void setMessageService(MessageService messageService) {
    this.messageService = messageService;
  }
}
```

### @Qualifier

If there are multiple beans, we need use @Qualifier to tell Spring which one is correct.

```java
@Component
@Qualifier("SMSService")
public class SMSService implements MessageService {
}

@Component
@Qualifier("EmailService")
public class EmailService implements MessageService {
}
```

#### Field-Base Injection

```java
@Component
public class Client {
  @Autowired
  @Qualifier("SMSService")
  private MessageService messageService;
}
```

#### Constructor-Based Injection

```java
@Component
public class Client {
  private final MessageService messageService;
  
  @Autowired
  public Client(@Qualifier("SMSService") MessageService messageService) {
    this.messageService = messageService;
  }
}
```

#### Setter-Based Injection

```java
@Component
public class Client {
  private MessageService messageService;
  
  @Autowired(required = false)
  @Qualifier("SMSService")
  public void setMessageService(MessageService messageService) {
    this.messageService = messageService;
  }
}
```

### @Primary

We can also use @Primary to indicate the default one.

## Web Service



### @Controller



### @RestController

Instruct Spring that all the controller's actions are REST endpoints.
