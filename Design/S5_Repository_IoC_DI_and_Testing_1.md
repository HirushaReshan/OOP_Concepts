# S5: Repository Layer, IoC, Dependency Injection, and Testing

**Before this:** S4. OOP Phase 7 (dependencies and layered design) covers the ideas in plain Java; this guide is the Spring version, plus the testing lecture.
**Goal:** by the end you can read any starter's controller → service → repository wiring in a minute, explain beans, component scanning and constructor injection, fix the two common startup errors, and write unit tests (JUnit 5), mock-based tests (Mockito) and controller tests (MockMvc).
**Time needed:** about 2 hours.

**Built from your files:** *Introduction to Repository Layer, Inversion of Control, and Dependency Injection in Spring Boot* (25 pages), *Testing Spring Boot Applications using JUnit 5, Mockito and MockMvc* (24 pages), and the `repo` packages and `pom.xml` of your three practice starters.

> **Exam link.** The briefing: *"work within controller, service and repository layers"*. Lecture 6's mock briefing: *"the repository... read them, do not edit them. Constructor injection in the service and controller: you read it, you do not write it."* The papers also say "zip the complete project including any test files", and every starter `pom.xml` includes `spring-boot-starter-test`.

---

## Part A: The repository layer, IoC and DI

## 1. Four words (lecture)

| Word | Meaning | Example |
| :-- | :-- | :-- |
| **Dependency** | Something a class needs to do its job | `ProductController` needs a `ProductService` |
| **Bean** | An object Spring creates and looks after | Your `@Service`, `@RestController`, `@Repository` objects |
| **Container** | Where the beans live; also called the **application context**; filled when the app starts | Spring's box of objects |
| **Inject** | Hand a bean to a class, through its constructor | Spring passes the service into the controller; the class never calls `new` |

## 2. Field injection versus constructor injection

```java
// Field injection (Lecture 3 style)
@RestController
public class ProductController {
    @Autowired
    private ProductService service;
}
```

```java
// Constructor injection (use this)
@RestController
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }
}
```

| | Field injection | Constructor injection |
| :-- | :-- | :-- |
| How | Spring sets the private field **after** building the object | Spring calls the constructor with the bean |
| `final` allowed? | No | **Yes**: set once, never `null` |
| Dependencies visible? | Hidden in fields | **Listed in the constructor** |
| Usable without Spring (tests)? | No way in | `new ProductController(myService)` |
| Missing bean | Discovered late | **Fails at startup** |
| `@Autowired` needed? | Yes | **No**, if there is exactly one constructor (Spring 4.3+) |

Lecture: field injection is *"shortest to type, worst to live with"*; constructor injection is *"the Spring team's own recommendation"*. **All your starters use constructor injection.**

## 3. What a repository is

A **repository** is a class whose **only job is storage**: keep the objects, hand them back when asked. Typical operations: find all, find by id, save, delete.

- **Analogy (lecture): a library desk.** You ask; the librarian fetches. You do not know or care which shelf.

Today the data lives in memory (a list or map, *"like a whiteboard: wiped clean at the end of the day"*); real systems use a database (*"like a notebook: still there tomorrow"*). Keeping storage in one class means only that class changes when storage changes.

### 3.1 The lecture's version: an interface + an implementation

```java
public interface ProductRepository {                    // the contract: no annotation
    List<Product> findAllProducts();
    Optional<Product> findProductById(Long id);
    Product saveProduct(Product product);
    boolean deleteProduct(Long id);
    List<Product> findByName(String name);
}
```

```java
@Repository
public class InMemoryProductRepository implements ProductRepository {

    private final List<Product> products = new ArrayList<>();
    private Long nextId = 1L;

    @Override
    public Product saveProduct(Product product) {
        if (product.getId() == null) {
            product.setId(nextId++);
        }
        products.add(product);
        return product;
    }

    // findAllProducts, findProductById, deleteProduct, findByName: same idea
}
```

```java
@Service
public class ProductService {

    private final ProductRepository productRepository;      // the INTERFACE type

    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    public Product findById(Long id) {
        return productRepository.findProductById(id)
                .orElseThrow(() -> new ProductNotFoundException(id));
    }
}
```

> **Slide correction.** On "The Service Layer Asks for the Interface", the field is called `productRepository` but `findById` calls `repository.findById(id)`. Neither the name nor the method exists, so it does not compile. Use `productRepository.findProductById(id)`, as above. The lecture also lists `getAllSummaries()` inside the repository interface; summaries are business logic and belong in the service (S3).

### 3.2 Your starters' version: a concrete class around a `Map`

```java
@Repository
public class PassRepository {

    private final Map<Long, ParkingPass> passes = new ConcurrentHashMap<>();

    public PassRepository() {
        passes.put(1L, new HourlyPass(1, "CAR-101", 250));
        // TODO after Task 2: add new WeekendPass(2,"VAN-202",800).
    }

    public Collection<ParkingPass> all() { return List.copyOf(passes.values()); }
    public ParkingPass get(long id)      { return passes.get(id); }
    public boolean contains(long id)     { return passes.containsKey(id); }
    public void save(ParkingPass pass)   { passes.put(pass.getId(), pass); }
}
```

| Piece | Meaning |
| :-- | :-- |
| No interface | Simpler: the service depends on the class directly. Fine for a small exam project. |
| `Map<Long, ParkingPass>` | Look up by id in one step (OOP Phase 3 §5). |
| `ConcurrentHashMap` | A `HashMap` that is safe when several requests use it at the same moment (a web server handles requests in parallel). Used exactly like `HashMap`. |
| Seed data in the constructor | The starting objects. The `// TODO after Task 2` line is the one place you may edit here. |
| `all()` returns `List.copyOf(...)` | An **unmodifiable copy**: callers can read it but not add to or remove from it (trying throws `UnsupportedOperationException`). To sort it, use a stream (`.sorted(...)`), not `list.sort(...)`. |
| `get(id)` returns `null` if missing | So the **service** must check for `null` and throw the not-found exception. |
| `contains(id)` | For the duplicate-id check. |
| `save(pass)` | `put` replaces any existing entry with the same id, so check `contains` **before** saving. |
| **Order of `all()`** | `HashMap` keeps no order, so `GET /api/passes` may list items in any order. If a paper asks for deterministic order, sort in the service. |

The dispatch starter calls them `find`, `exists`, `save`; the scheduler starter only has `all` and `find`. **Read the method names before writing the service.**

## 4. Inversion of Control and what Spring does at startup

| Traditional control | Inversion of Control |
| :-- | :-- |
| Your code calls `new` when it chooses | The container creates the objects and holds them |
| Your code builds the whole object graph | It is assembled before your first line runs |
| A library is something you call | A framework calls you |

*The Hollywood Principle: don't call us, we'll call you.*

What happens between pressing Run and "Started Application" (lecture):

1. **Component scan:** Spring walks every package below the `@SpringBootApplication` class, looking for labels.
2. **Bean definitions registered:** each `@Component`, `@Service`, `@Repository`, `@RestController`, `@RestControllerAdvice` class.
3. **Dependencies resolved:** for each bean, Spring reads the constructor parameters and finds a bean of the matching **type**.
4. **Context ready:** objects are created in dependency order (repository → service → controller) and wired. Only then does Tomcat accept requests.

**You never call `new` on a bean.**

### 4.1 The four labels

| Annotation | Goes on | Says |
| :-- | :-- | :-- |
| `@Component` | Any class, when nothing more specific fits | "Make an object of this class." |
| `@Service` | Business-rule classes | Same as `@Component`, named for readers. |
| `@Repository` | Storage classes (the **implementation**, never the interface) | "This class stores data." |
| `@RestController` | Web classes | "Answers web requests; return values become JSON." |

The interface gets **no label**: Spring cannot make an object of an interface.

### 4.2 Singleton scope

**Bean scope** = how many objects Spring makes of a class. The default is **singleton**: **one** object, shared by everyone who asks. That is why a pass you POST is still there on the next GET: one repository, one map. (`@Scope("prototype")` makes a new one each time; you will not need it.)

The danger (OOP Phase 7 §7.3): never keep one user's private data in a field of a service, because every user shares that one object.

### 4.3 Swapping implementations, and `@Primary`

If the service depends on an interface, swapping storage means adding one new class (lecture's `DatabaseProductRepository`). No other file changes.

If **two** classes implement the interface and both have `@Repository`:

```
Parameter 0 of constructor in ProductService required a single bean, but 2 were found
```

Fix: delete the one you do not need, or put **`@Primary`** on the one Spring should choose.

### 4.4 Startup errors to recognise

| Message | Cause | Fix |
| :-- | :-- | :-- |
| `required a bean of type '...' that could not be found` | Missing `@Service`/`@Repository`, or the class is outside the main package | Add the label; move the file |
| `required a single bean, but 2 were found` | Two implementations | Delete one, or `@Primary` |
| `Port ... was already in use` | Old run still going | Stop it |

Lecture exercise: delete `@Repository` from the implementation, restart, read the error, put it back.

### 4.5 Which layer? (lecture)

| Responsibility | Layer |
| :-- | :-- |
| Converting a `Product` to JSON | `@RestController` boundary (Jackson) |
| Rejecting a negative price | `@Service` |
| Generating the next id | `@Repository` (storage owns identity). In the papers, the client sends the id and the service validates it. |
| Turning the exception into 404 | `@ControllerAdvice` |
| Applying a 20% discount cap | `@Service` (or the model) |
| Choosing ArrayList or database | `@Repository` |

**Rule: each layer only talks to the layer directly below it.**

---

## Part B: Testing

## 5. Four words (lecture)

| Word | Meaning |
| :-- | :-- |
| **Unit test** | A small automatic check that runs one piece of code and checks the answer. One class, no server, milliseconds. |
| **Assertion** | The check itself: compares expected with actual. No assertion = nothing checked. |
| **Mock** | A stand-in object, "like a crash-test dummy". Lets you test the service without the real repository. |
| **Coverage** | The percentage of lines your tests ran. It shows what ran, not what was checked. |

Why test (lecture): catch bugs before users do; a test's name documents the rule; change code without fear; tests are evidence of quality in your coursework.

## 6. Where tests live, and how to create one

```
src/main/java/edu/practice/parking/service/ParkingService.java       <- code
src/test/java/edu/practice/parking/service/ParkingServiceTest.java   <- its test
```

Same package, `src/test/java` instead of `src/main/java`, class name + `Test`. Tests are never part of the running app.

IntelliJ steps (lecture):

1. Open the class to test; click on its name.
2. **Ctrl + Shift + T** (Mac: Cmd + Shift + T) → **Create New Test...** (or right-click the package under `src/test/java` → New → Java Class).
3. Choose **JUnit 5** → OK.
4. Write methods with `@Test`. If anything is red: **Alt + Enter** → Import.
5. Click the green arrow next to the class (all tests) or a method (one test). Green tick = pass; red cross = fail.

If `src/test/java` does not exist yet: right-click `src` → New → Directory → choose `test/java` from IntelliJ's suggestions.

## 7. JUnit 5 fundamentals

### 7.1 Arrange, Act, Assert

Every test has three parts:

1. **Arrange:** build what the test needs.
2. **Act:** call **one** method, the behaviour under test.
3. **Assert:** check the outcome.

Name tests `shouldDoXWhenY`: the name is the specification. **One behaviour per test.**

### 7.2 A real unit test for the parking solution

```java
package edu.practice.parking.service;

import edu.practice.parking.dto.NewHourlyPass;
import edu.practice.parking.dto.Quote;
import edu.practice.parking.error.PassNotFoundException;
import edu.practice.parking.repo.PassRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

class ParkingServiceTest {

    private ParkingService service;

    @BeforeEach
    void setUp() {
        service = new ParkingService(new PassRepository());     // no Spring needed
    }

    @Test
    void shouldChargeTwoHoursFor61Minutes() {
        Quote quote = service.quote(1, 61);
        assertEquals(500, quote.priceCents());
    }

    @Test
    void shouldThrowWhenPassMissing() {
        assertThrows(PassNotFoundException.class, () -> service.quote(99, 61));
    }

    @Test
    void shouldRejectZeroMinutes() {
        assertThrows(IllegalArgumentException.class, () -> service.quote(1, 0));
    }

    @Test
    void shouldPutCheaperPassFirst() {
        List<Quote> quotes = service.allQuotes(481);
        assertEquals(2L, quotes.get(0).id());
    }

    @Test
    void shouldNotSaveInvalidPass() {
        int before = service.all().size();
        assertThrows(IllegalArgumentException.class,
                () -> service.addHourly(new NewHourlyPass(3, "   ", 300)));
        assertEquals(before, service.all().size());
    }
}
```

| Piece | Meaning |
| :-- | :-- |
| `class ParkingServiceTest` | A normal class; no `public` needed in JUnit 5. |
| `@BeforeEach` | Runs before **every** test: each test gets a fresh service and repository. |
| `@Test` | JUnit runs this method. No `main` needed. |
| `import static ...Assertions.assertEquals` | `static` import: lets you write `assertEquals(...)` instead of `Assertions.assertEquals(...)`. |
| `assertEquals(expected, actual)` | **Expected first**, then actual. |
| `assertThrows(Type.class, () -> code)` | Passes only if the lambda throws that type. |
| Constructor injection pays off | `new ParkingService(new PassRepository())`: the test builds its own objects. |

### 7.3 The four assertions

| Assertion | Use |
| :-- | :-- |
| `assertEquals(expected, actual)` | Right value? For doubles add a margin: `assertEquals(349.99, p.getPrice(), 0.01)`. |
| `assertThrows(X.class, () -> ...)` | Failed the right way? |
| `assertTrue(condition)` | A yes/no rule. Prefer `assertEquals` when there is a value. |
| `assertNotNull(x)` | Did anything come back? Never the only check. |

### 7.4 Red before green

Break the code on purpose (change `500` to `501` in a test, or `+1` in the code). If the test stays green, it never tested anything. A failing test prints, e.g.:

```
org.opentest4j.AssertionFailedError: expected: <500> but was: <250>
```

Read it: **expected** what you said, **but was** what the code returned.

## 8. Mockito: fake collaborators

A real repository cannot answer two questions (lecture): *what if saving fails?* and *did the service call save?* A **mock** can.

```java
package edu.practice.parking.service;

import edu.practice.parking.dto.NewHourlyPass;
import edu.practice.parking.repo.PassRepository;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.never;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
class ParkingServiceMockTest {

    @Mock
    private PassRepository repo;

    @InjectMocks
    private ParkingService service;

    @Test
    void shouldNotSaveWhenIdAlreadyExists() {
        when(repo.contains(1L)).thenReturn(true);                       // Arrange: script the fake

        assertThrows(IllegalArgumentException.class,                    // Act + Assert
                () -> service.addHourly(new NewHourlyPass(1, "CAR-1", 100)));

        verify(repo, never()).save(any());                              // Assert: save was NOT called
    }

    @Test
    void shouldSaveValidPass() {
        when(repo.contains(3L)).thenReturn(false);

        service.addHourly(new NewHourlyPass(3, " CAR-303 ", 300));

        verify(repo).save(any());                                       // save WAS called
    }
}
```

| Piece | Meaning |
| :-- | :-- |
| `@ExtendWith(MockitoExtension.class)` | Switch Mockito on for this test class. |
| `@Mock` | "Make me a fake." Every method does nothing and returns `null`/`0`/`false` until told otherwise. |
| `@InjectMocks` | "Make the **real** class and pass the fakes into its constructor." |
| `when(x).thenReturn(y)` | Arrange: "when the service asks this, answer that". |
| `when(x).thenThrow(e)` | Arrange: "throw this instead". |
| `verify(mock).method(args)` | Assert: the service **did** call it. |
| `verify(mock, never()).method(any())` | Assert: it did **not**. |
| `any()` | Any argument. |

Lecture's rule: **mock the collaborators, never the class you are testing.** Memory trick: `when` comes before the Act; `verify` comes after.

By default `MockitoExtension` is strict: a `when(...)` that the test never uses makes the test fail with `UnnecessaryStubbingException`. Only stub what the code under test will call.

## 9. MockMvc: testing the web layer without a server

```java
package edu.practice.parking.controller;

import edu.practice.parking.error.PassNotFoundException;
import edu.practice.parking.model.HourlyPass;
import edu.practice.parking.service.ParkingService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest(ParkingController.class)
class ParkingControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ParkingService service;

    @Test
    void shouldReturn404WhenPassMissing() throws Exception {
        when(service.quote(99, 61)).thenThrow(new PassNotFoundException(99));

        mockMvc.perform(get("/api/passes/99/quote").param("minutes", "61"))
                .andExpect(status().isNotFound())
                .andExpect(jsonPath("$.code").value("NOT_FOUND"));
    }

    @Test
    void shouldReturn201WhenPassCreated() throws Exception {
        when(service.addHourly(any())).thenReturn(new HourlyPass(3, "CAR-303", 300));

        mockMvc.perform(post("/api/passes/hourly")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("{\"id\":3,\"registration\":\"CAR-303\",\"rateCents\":300}"))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(3));
    }
}
```

| Piece | Meaning |
| :-- | :-- |
| `@WebMvcTest(ParkingController.class)` | Load **only** the web layer: this controller, the `@RestControllerAdvice` handler and JSON. No services, no repositories, no Tomcat. |
| `@Autowired MockMvc mockMvc` | A fake browser that sends requests straight into Spring. |
| `@MockBean ParkingService service` | Put a Mockito mock of the service into the context. |
| `perform(get(...).param("minutes", "61"))` | Send `GET /api/passes/99/quote?minutes=61`. |
| `.andExpect(status().isNotFound())` | Expect 404. Also `isOk()`, `isCreated()`, `isBadRequest()`, `isConflict()`. |
| `jsonPath("$.code").value("NOT_FOUND")` | Look inside the JSON: `$` is the root object, `$.code` its `code` field. `$[0].id` = the first element's id in an array. |
| `\"` inside the string | An escaped double quote: a `"` character inside Java text. |
| `throws Exception` | `perform` can throw checked exceptions; the test simply passes them on. |

**Version note.** Your practice starters use Spring Boot **3.3.5**, where the annotation is **`@MockBean`** (package `org.springframework.boot.test.mock.mockito`). The lecture slides use Spring Boot **4**, where it is **`@MockitoBean`** (package `org.springframework.test.context.bean.override.mockito`), `@MockBean` no longer exists, the test starter is `spring-boot-starter-webmvc-test`, and `@WebMvcTest` lives in a different package. If an import is red, delete it and let IntelliJ's **Alt + Enter** find the right one for your version.

## 10. Three kinds of test, and coverage

| Kind | Annotation | Speed | Proves |
| :-- | :-- | :-- | :-- |
| Unit | none, or `@ExtendWith(MockitoExtension.class)` | milliseconds | One class's logic |
| Slice | `@WebMvcTest` | ~1–2 s | URLs, status codes, JSON, the error handler |
| Full context | `@SpringBootTest` | seconds+ | The whole wiring |

**The testing pyramid:** many fast unit tests at the bottom, a few slice tests, very few full tests, and manual end-to-end checks (Postman, Swagger) at the top. *Test each rule once, at the lowest layer that can see it.*

**Coverage:** right-click the test class → **Run with Coverage** (in newer IntelliJ: under *More Run/Debug*). Green lines ran, red lines never ran, yellow = only one side of an `if` ran. Write tests for the red error paths. Aim above 70% of the service, then stop chasing the number.

**In the timed exam,** tests earn no marks of their own in the published scheme. If you write any, make sure they compile, because a test file that does not compile can stop IntelliJ building the project. Running your app and checking the six Section B behaviours by hand is the priority.

---

## 11. Practice questions

| # | Question | Answer |
| :-- | :-- | :-- |
| 1 | Who creates the service in a Spring Boot app? | The application context, at startup. |
| 2 | What is inverted in IoC? | Control over object creation, moved to the container. |
| 3 | Why can the field be `final` with constructor injection? | It is assigned in the constructor, during creation. |
| 4 | Your starter repository's `all()` returns `List.copyOf(...)`. Can you call `.sort()` on it? | No: it is unmodifiable. Sort with a stream. |
| 5 | `repo.get(99)` in the parking starter returns? | `null`. The service must throw `PassNotFoundException`. |
| 6 | Two `@Repository` classes implement one interface. Result? | Startup failure: "required a single bean, but 2 were found". |
| 7 | What are the three parts of a test? | Arrange, Act, Assert. |
| 8 | `@Mock` vs `@InjectMocks`? | Fake collaborator vs the real class under test with the fakes injected. |
| 9 | Spring Boot 3.3.5: `@MockBean` or `@MockitoBean`? | `@MockBean`. |
| 10 | What does `jsonPath("$.code")` read? | The `code` field of the JSON response body. |

---

## 12. Cheat sheet

```java
// wiring: read it, do not change it in the exam
@Service
public class XService {
    private final XRepository repo;
    public XService(XRepository repo) { this.repo = repo; }
}

// unit test
class XServiceTest {
    private XService service;
    @BeforeEach void setUp() { service = new XService(new XRepository()); }
    @Test void shouldDoXWhenY() { assertEquals(expected, service.doSomething()); }
    @Test void shouldThrowWhenMissing() { assertThrows(XNotFoundException.class, () -> service.get(99)); }
}

// mock test
@ExtendWith(MockitoExtension.class)
class XServiceMockTest {
    @Mock XRepository repo;
    @InjectMocks XService service;
    @Test void shouldNotSaveInvalid() {
        assertThrows(IllegalArgumentException.class, () -> service.add(badInput));
        verify(repo, never()).save(any());
    }
}

// controller slice test (Boot 3.x: @MockBean; Boot 4: @MockitoBean)
@WebMvcTest(XController.class)
class XControllerTest {
    @Autowired MockMvc mockMvc;
    @MockBean XService service;
    @Test void shouldReturn404() throws Exception {
        when(service.get(99)).thenThrow(new XNotFoundException(99));
        mockMvc.perform(get("/api/x/99")).andExpect(status().isNotFound());
    }
}
```

**Next: S6, the exam toolkit and full worked solutions to your three practice papers.**
