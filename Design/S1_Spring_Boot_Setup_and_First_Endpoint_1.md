# S1: Spring Boot Setup and Your First Endpoint

**Module:** 5COSC019W, enterprise / design lecture stream
**Before this:** OOP Phases 0 to 8 (classes, inheritance, polymorphism, exceptions, layers).
**Goal:** by the end you can explain what an enterprise application is, create or open a Spring Boot project, understand every part of `pom.xml`, run the app, write a GET endpoint, test it, and open an exam starter project without panic.
**Time needed:** about 1.5 hours with the practical steps.

**Built from your files:** *Introduction to Enterprise Application Development Using Spring Boot* (32 pages), plus the `pom.xml`, `README.md` and `application.properties` of your three practice starters.

---

## The Spring track roadmap

| Guide | Title | Lecture it covers | Exam task |
| :-- | :-- | :-- | :-- |
| **S1** | **Spring Boot setup and first endpoint** | Introduction to Enterprise Application Development | Opening and running the starter (needed for Section B) |
| S2 | REST, JSON, Jackson, records and Postman | Building REST APIs: Encapsulation, Jackson, Postman | Task 4 |
| S3 | Service layer, inheritance and polymorphism in Spring | Service Layer; Inheritance in Spring Boot; Polymorphism in Spring Boot | Tasks 1, 2, 3 |
| S4 | Error handling and status codes | API Error Handling | Task 3 |
| S5 | Repository, IoC, DI and testing | Repository Layer, IoC and DI; Testing with JUnit 5, Mockito, MockMvc | Reading the starter; "any test files" |
| S6 | Exam toolkit and full solutions to the three practice papers | Your three practice papers | All tasks, Section B |

---

## 1. What is an enterprise application?

An **enterprise application** is a large software system that runs the core work of an organisation: a bank's money transfers, a shop's orders, a hospital's appointments.

What makes it different from a small program (lecture slide):

| Feature | Meaning |
| :-- | :-- |
| **Multi-user** | Thousands of people use it at the same time without it crashing. |
| **Massive data** | Huge amounts of data, many transactions per second. |
| **Secure and reliable** | Must almost never be down, must protect data, must check who is allowed to do what. |
| **Integrated** | Connects to databases, other companies' systems and older systems. |

Lecture's key insight: *enterprise applications are not just more code; they are built with strict architecture and design patterns so teams can maintain them for decades.*

- **Analogy:** a garden shed versus a hospital. You can build a shed any way you like. A hospital needs plans, separate wards, fire exits and rules, because many people depend on it for years.

---

## 2. Multi-layer architecture

Enterprise apps split the work into **layers**. A layer is a group of classes with one kind of job.

| Layer (lecture name) | Job | In your Spring project |
| :-- | :-- | :-- |
| **1. Presentation layer** | Receives requests from users or other programs; returns responses | The **controller** (`@RestController`) |
| **2. Application layer** | Business rules: calculations, decisions, workflows | The **service** (`@Service`) |
| **3. Data layer** | Stores and fetches data | The **repository** (`@Repository`) |

Lecture's quick question: *if a bank calculates mortgage interest, which layer handles that code?* **The application (business) layer, i.e. the service.**

You met this in OOP Phase 7. The Spring lectures build it step by step: controller only (S1, S2), then a service (S3), then a repository (S5).

---

## 3. What Spring and Spring Boot are

| Word | Plain meaning |
| :-- | :-- |
| **Framework** | Ready-made code that runs your program and calls **your** code at the right moments. (A **library** is the opposite: you call it.) |
| **Spring** | A Java framework for building enterprise applications. Really a family ("ecosystem") of tools. |
| **Spring Boot** | The quick way to start a Spring application: sensible defaults, almost no setup, and a web server built in. |
| **Spring Data / Spring Security** | Other family members (databases; logins and permissions). Not needed for your exam. |

What Spring Boot gives you (lecture slide):

| Feature | Meaning |
| :-- | :-- |
| **Embedded server** | A web server called **Tomcat** is inside your app. You do not install a server; you just run `main`. |
| **Auto-configuration** | Spring Boot looks at what libraries you added and configures them for you. |
| **Convention over configuration** | Sensible defaults; you change only what you need. |
| **No XML config** | Older Spring used long XML files. Spring Boot uses Java and annotations. |

- **Analogy:** a furnished flat. Plain Java is an empty flat: you buy and assemble everything. Spring Boot arrives furnished; you only move the furniture you dislike.

A **web server** is a program that waits for requests arriving over the network and sends back responses. **Tomcat** is the one Spring Boot uses. It listens on a **port**: a numbered door on your computer. Default: **8080**. Your practice starters use **8081, 8082 and 8083**.

---

## 4. Maven and `pom.xml`

### 4.1 What Maven does

A **dependency** (in this sense) is a library your project needs, such as Spring Web. **Maven** is a build tool that:

1. reads a list of dependencies from `pom.xml`,
2. downloads them from the internet (from "Maven Central") into a cache on your computer (`~/.m2`),
3. compiles your code and runs it.

- **Analogy:** a shopping list handed to a delivery service. You write what you need; Maven fetches it.

### 4.2 Reading a real `pom.xml` (the parking starter, reformatted)

```xml
<project ...>
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.5</version>
        <relativePath/>
    </parent>

    <groupId>edu.practice</groupId>
    <artifactId>parking-practice</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

| Part | Meaning |
| :-- | :-- |
| XML | A text format of nested `<tag>...</tag>` pairs. `pom.xml` is written in it. |
| `<parent>` ... `spring-boot-starter-parent` `3.3.5` | Inherit Spring Boot's standard settings, version **3.3.5**. This one line fixes compatible versions for all Spring libraries. |
| `<groupId>`, `<artifactId>`, `<version>` | Your project's "address": organisation, project name, version. `SNAPSHOT` in a version means "still in development". |
| `<java.version>17</java.version>` | Compile for Java 17. |
| `spring-boot-starter-web` | The main dependency: **Tomcat** + **Spring MVC** (the part that handles web requests) + **Jackson** (JSON). A **starter** is a bundle of libraries that work together. |
| `spring-boot-starter-test` with `<scope>test</scope>` | JUnit 5, Mockito, AssertJ, Spring Test. `test` scope = only used by test code, never by the running app. |
| No `<version>` on the dependencies | The parent decides the versions. |
| `spring-boot-maven-plugin` | Lets Maven run the app (`mvn spring-boot:run`) and package it as a runnable jar. |

**Version note.** Your lecture slides say *Spring Boot 4.x*. Your practice starters use **3.3.5**. For everything in this module the code is the same; the only difference you will meet is in tests (S5: `@MockBean` in 3.x versus `@MockitoBean` in newer versions). **In the exam, use whatever the supplied `pom.xml` says. Never change the versions.**

---

## 5. Creating a new project (lecture walkthrough)

### 5.1 With Spring Initializr

**Spring Initializr** is a website, `start.spring.io`, that generates an empty Spring Boot project for you.

1. Open `start.spring.io`.
2. **Project:** Maven. **Language:** Java. **Spring Boot:** the version your lecturer uses.
3. **Group:** `uk.ac.westminster`. **Artifact:** `products-api`. (These become your package name.)
4. **Packaging:** Jar. **Java:** 17 (or the version you have installed).
5. **Add dependency:** Spring Web.
6. **Generate**, unzip, then in IntelliJ: **File → Open** → choose the folder that contains `pom.xml` → wait for Maven to finish downloading (progress bar at the bottom).

### 5.2 By hand (lecture's manual route)

1. IntelliJ: **New Project → Maven**.
2. Write `pom.xml` as in Section 4.2.
3. Create the main class (Section 6).
4. Create `src/main/resources/application.properties` (Section 7).
5. Run `main`.

### 5.3 The folder layout every project uses

```
parking-practice-starter/            <- open THIS folder in IntelliJ (it has pom.xml)
├── pom.xml
├── README.md
└── src
    ├── main
    │   ├── java/edu/practice/parking/
    │   │   ├── ParkingApplication.java     <- the main class
    │   │   ├── controller/                 <- web layer
    │   │   ├── service/                    <- business rules
    │   │   ├── repo/                       <- storage
    │   │   ├── model/                      <- your OOP classes
    │   │   ├── dto/                        <- request/response shapes (S2)
    │   │   └── error/                      <- exceptions + handler (S4)
    │   └── resources/
    │       └── application.properties      <- settings
    └── test/java/...                       <- tests (S5); may not exist yet
```

The folders under `edu/practice/parking/` are **packages** (OOP Phase 2). After building, Maven also creates a `target/` folder with compiled files; you never edit it and do not need to submit it.

---

## 6. The main class, line by line

```java
package edu.practice.parking;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ParkingApplication {

    public static void main(String[] args) {
        SpringApplication.run(ParkingApplication.class, args);
    }
}
```

| Line | Meaning |
| :-- | :-- |
| `package edu.practice.parking;` | This class lives in the **top** package. Everything else sits in packages **below** it. That matters for scanning (below). |
| `@SpringBootApplication` | An **annotation** (a label starting with `@`) that combines three: `@SpringBootConfiguration` (this is a configuration class), `@EnableAutoConfiguration` (switch on auto-configuration), `@ComponentScan` (search this package and every package below it for labelled classes). |
| `public static void main(String[] args)` | The ordinary Java entry point (Pre-Phase 0). |
| `SpringApplication.run(ParkingApplication.class, args);` | Start Spring: create the application context, scan for components, build and wire objects, start Tomcat. `ParkingApplication.class` means "the class itself" as a value. |

**Component scanning** = Spring searching packages for classes labelled `@RestController`, `@Service`, `@Repository`, `@RestControllerAdvice` and so on, and creating one object of each. **A class in a package that is not below the main class's package is never found.** (When you create a new class in the exam, put it in the right package folder.)

---

## 7. `application.properties`

`src/main/resources/application.properties` holds settings as `key=value` lines. The parking starter:

```properties
server.port=8081
spring.application.name=parking-practice
```

| Setting | Meaning |
| :-- | :-- |
| `server.port=8081` | Tomcat listens on port 8081, so URLs start `http://localhost:8081`. |
| `spring.application.name` | A name for the app, shown in logs. |

`localhost` means "this computer". So `http://localhost:8081/api/passes` means "the program on my own computer, door 8081, path `/api/passes`".

> **Exam habit:** open `application.properties` first to find the port. Many "it doesn't work" moments are just the wrong port in the URL.

---

## 8. Running the app

Click the green triangle next to `main`, or run `mvn spring-boot:run` in a terminal in the project folder.

A successful start ends with lines like:

```
Tomcat started on port 8081 (http) with context path '/'
Started ParkingApplication in 1.9 seconds
```

The lecture calls this **the first milestone**: an embedded web server is listening.

Open `http://localhost:8080` (or your port) with no endpoint written yet and you see a **Whitelabel Error Page** with status 404. That is good news: Spring is running; there is simply nothing at that path.

### 8.1 The four buttons

| Button | Use |
| :-- | :-- |
| Green triangle | Run. |
| Red square | Stop. |
| Circular arrow (Rerun) | Stop and start again. |
| Bug icon | Debug (run with breakpoints). |

> **Restart after every change to Java code.** A running app keeps using the old code. "My new endpoint returns 404" usually means you did not restart.

### 8.2 Startup failures you will meet

| Console says | Cause | Fix |
| :-- | :-- | :-- |
| `Port 8081 was already in use` | An earlier run is still going. | Press the red square on the old run (check all Run tabs), then run again. |
| `Parameter 0 of constructor in X required a bean of type 'Y' that could not be found` | A class is missing its `@Service`/`@Repository` label, or is outside the main package. | Add the label; move the file into the right package. |
| `java: cannot find symbol` / red code | A compile error. | Fix it first; Spring cannot start code that does not compile. |
| `release version 17 not supported` | IntelliJ is using an older Java. | File → Project Structure → SDK: choose 17 or newer. |
| Dependencies red, `Cannot resolve symbol 'springframework'` | Maven has not downloaded the libraries. | Open the Maven tool window → **Reload All Maven Projects**. The lab must have them cached or internet access. |

---

## 9. Your first endpoint

### 9.1 What an endpoint is

A **REST endpoint** is one URL + one HTTP method that your app answers. "GET `/hello`" is an endpoint.

```
 Browser / client                     Spring Boot
 ---------------------  GET /hello ->  method runs
                        <- "Hello!"    return value = response body, status 200
```

| Part | Meaning |
| :-- | :-- |
| **HTTP** | The language browsers and servers use to talk. |
| **Request** | The message sent to the server: a **method** (GET, POST...) and a **path** (`/hello`). |
| **Response** | The answer: a **status code** (200 = OK) and a **body** (the content). |
| **GET** | "Give me something." Changes nothing. |

### 9.2 The lecture's live code

```java
package uk.ac.westminster.productsapi;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDate;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello from Spring Boot!";
    }

    @GetMapping("/status")
    public String status() {
        return "API running - " + LocalDate.now().toString();
    }
}
```

| Line | Meaning |
| :-- | :-- |
| `@RestController` | "Spring, this class handles web requests. Create one object of it at startup (component scanning), and send each method's return value back as the response body." |
| `@GetMapping("/hello")` | "When a **GET** request arrives for path `/hello`, run this method." |
| `public String hello()` | An ordinary method. It must be `public`. |
| `return "Hello from Spring Boot!";` | This text **is** the response body. Spring sets status 200 automatically. |
| `LocalDate.now()` | Today's date (from `java.time`). |

### 9.3 Test it (lecture steps)

1. Save `HelloController.java` (in a package below the main class).
2. Stop the app (red square), run again (green triangle).
3. Open the browser at `localhost:8080/hello`.
4. You see: `Hello from Spring Boot!`

Lecture question: *if I add `@GetMapping("/goodbye")` returning `"Goodbye!"`, which URL answers?* → `http://localhost:8080/goodbye`.

A browser can only send GET requests from its address bar. For POST and the rest you need Swagger UI, Postman, curl or IntelliJ's HTTP Client (S2).

---

## 10. Swagger UI

**Swagger UI** is a web page that lists every endpoint in your app and lets you send requests by clicking. It is generated automatically by a library called **springdoc-openapi**. **OpenAPI** is the standard format for describing an API.

Add to `<dependencies>` in `pom.xml`:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

Then: save `pom.xml` → reload Maven → restart → open `http://localhost:8080/swagger-ui.html`.

Using it: click an endpoint → **Try it out** → fill in any values → **Execute** → read the **response body**, the **status code** and the **headers**.

> **Important for your practice papers:** the three starter `pom.xml` files do **not** include springdoc, so `/swagger-ui.html` will give 404 there. Test them with Postman, curl or IntelliJ's HTTP Client (S2 shows all three). Do not edit the exam `pom.xml` to add Swagger unless allowed: if the download fails in the lab, your project may stop building. Also note that springdoc 2.3.0 is matched to Spring Boot 3.2-era versions; newer Boot versions need a newer springdoc.

---

## 11. Version control with Git

The lecture insists on Git from day one because the **coursework** requires evidence of development history.

| Word | Meaning |
| :-- | :-- |
| **Git** | A tool that records snapshots ("commits") of your project, so you can go back. |
| **Repository (repo)** | A project tracked by Git. (Different from Spring's `@Repository`.) |
| **Commit** | One saved snapshot with a message. |
| **GitHub** | A website that stores Git repos online. |

```bash
git init                                  # 1. start tracking this folder (once)
git add .                                 # 2. stage every changed file
git commit -m "Tutorial 1: Spring Boot project created, /hello endpoint working"   # 3. snapshot
git remote add origin https://github.com/[username]/products-api.git   # 4. link GitHub (once)
git branch -M main
git push -u origin main                   # 5. upload
git push                                  # later uploads
```

Good messages say **what** changed: "Lecture 2: Product class with encapsulation". Bad: "fix", "asdf", "update 3". A `.gitignore` file tells Git to ignore `target/`.

Git is for your coursework, not for the timed lab exam.

---

## 12. Practical: open and run an exam-style starter

Do this now with `SIMPLE_Starter_Project.zip` from your practice pack. It is the exact skill the briefing lists: *practise importing an existing pom.xml, launching an application, reading error logs*.

1. Unzip `SIMPLE_Starter_Project.zip`. Inside is `parking-practice-starter/`.
2. IntelliJ → **File → Open** → select the `parking-practice-starter` folder (the one containing `pom.xml`) → **Open as Project**. If asked, choose **Trust Project**.
3. Wait until the Maven progress bar at the bottom finishes. If code is red, open the **Maven** tool window (right edge) and click **Reload All Maven Projects**.
4. Open `src/main/resources/application.properties`: the port is **8081**.
5. Open `ParkingApplication.java` and click the green triangle next to `main`.
6. Look for `Tomcat started on port 8081` in the console.
7. Browser: `http://localhost:8081/api/passes`. You should see JSON for one hourly pass, something like this (the order of the fields may differ):

```json
[{"id":1,"registration":"CAR-101","rateCents":250}]
```

8. Try `http://localhost:8081/api/passes/1/quote?minutes=61`. You get an error, because that endpoint is still a `// TODO`. That is Task 4.
9. Stop the app.

If step 7 works, you can do Section B testing in the exam.

> **Why does the starter run at all when methods `throw new UnsupportedOperationException("Task 1")`?** Those lines compile fine. They only fail when called. The starter is designed to start from minute one.

---

## 13. Knowledge check (lecture's own questions, plus a few)

| # | Question | Answer |
| :-- | :-- | :-- |
| 1 | Which layer evaluates business rules and workflows? | The application layer (the service). |
| 2 | Which annotation tells Spring to detect a class and handle HTTP REST requests? | `@RestController`. |
| 3 | Which URL opens Swagger UI once springdoc is added? | `/swagger-ui.html`. |
| 4 | What does `@SpringBootApplication` combine? | `@SpringBootConfiguration`, `@EnableAutoConfiguration`, `@ComponentScan`. |
| 5 | What does `spring-boot-starter-web` bring? | Embedded Tomcat, Spring MVC and Jackson. |
| 6 | Where do you find the port? | `application.properties`, `server.port`. |
| 7 | You added an endpoint but get 404. First thing to check? | Did you restart the app? Then: the path, the port, the package. |
| 8 | Why do the practice starters not show Swagger UI? | Their `pom.xml` has no springdoc dependency. |

---

## 14. Glossary

| Term | Meaning |
| :-- | :-- |
| **Annotation** | A label beginning with `@` that tells Java or a framework something about a class, method or parameter. |
| **Application context** | Spring's container holding all the objects it created. |
| **Auto-configuration** | Spring Boot setting things up based on the libraries present. |
| **Component scan** | Spring searching packages below the main class for labelled classes. |
| **Dependency (Maven)** | A library your project needs, listed in `pom.xml`. |
| **Embedded server** | A web server packaged inside your app (Tomcat). |
| **Endpoint** | One HTTP method + path that your app answers. |
| **Framework** | Code that runs your program and calls your code. |
| **HTTP** | The request/response protocol of the web. |
| **localhost** | This computer. |
| **Maven** | Build tool: downloads dependencies, compiles, runs. |
| **`pom.xml`** | Maven's project file. |
| **Port** | A numbered "door" a server listens on (8080, 8081...). |
| **Spring Boot** | Fast, pre-configured way to build Spring apps. |
| **Starter** | A bundle of compatible libraries (`spring-boot-starter-web`). |
| **Swagger UI** | A generated web page for trying your endpoints. |
| **Tomcat** | The web server inside Spring Boot. |
| **Whitelabel Error Page** | Spring's default error page (e.g. 404 when no endpoint matches). |

---

## 15. Cheat sheet

```java
@SpringBootApplication                       // on the main class, in the TOP package
public class App {
    public static void main(String[] args) { SpringApplication.run(App.class, args); }
}

@RestController                              // a web class, found by component scan
public class HelloController {
    @GetMapping("/hello")                    // GET /hello runs this
    public String hello() { return "Hello"; }   // return value = response body, 200 OK
}
```

| Remember | |
| :-- | :-- |
| Open the folder containing `pom.xml` | Wait for Maven; reload if red |
| Port is in `application.properties` | Practice starters: 8081 / 8082 / 8083 |
| Restart after every Java change | 404 on a new endpoint = probably not restarted |
| Classes must be below the main class's package | Otherwise never scanned |
| Do not change the exam `pom.xml` | Versions are fixed for you |

**Next: S2, REST, JSON, Jackson, records and Postman.**
