---
title: Java
date: 2023-02-09 17:14:22
categories:
  - [计算机]
---

.

<!-- more -->

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

## Maven Java Projects

Maven is a tool for building and managing your Java-based projects.

### Create a Maven Project

Step 1:

In VSCode, press `Ctrl+Shift+P`. Search for Java, Select `Java: Create Java Project`.

Alternatively, you may click `+` under the **Java Project Panel**.

Step 2: Choose `Maven`.

Step 3: Select `maven-archetype-quickstart`

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
