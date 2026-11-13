# Phase 7: Dependencies and Layered Design

**Module:** 5COSC019W Object Oriented Programming
**Before this:** Phase 6 (exceptions). Uses interfaces from Phase 5 and the service class from Phase 3.
**Goal:** by the end you can explain what a dependency is, write constructor injection, hide storage behind a repository interface, swap one implementation for another without touching other classes, and read the controller → service → repository layering in any project.
**Time needed:** about 1.5 hours.

**Built from your files:** Lecture 7 (Spring IoC & Dependency Injection).

> **Why this phase matters for the exam.** Lecture 6's mock briefing: *"The abstract parent class, the repository and ErrorResponse: read them, do not edit them. Constructor injection in the service and controller: you read it, you do not write it."* So you will not have to build this structure in the exam, but you **must be able to read it** within the first five minutes, to know where your code goes and which repository methods your service can call.

---

## Where we are on the roadmap

| Phase | Title | Status |
| :-- | :-- | :-- |
| Pre-Phase 0 | Absolute Java Basics | Done |
| Phase 1 | Classes and Objects | Done |
| Phase 2 | Encapsulation and UML Class Diagrams | Done |
| Phase 3 | Collections and Generics | Done |
| Phase 4 | Inheritance | Done |
| Phase 5 | Polymorphism and Abstraction | Done |
| Phase 6 | Exceptions | Done |
| **Phase 7** | **Dependencies and Layered Design** | **You are here** |
| Phase 8 | Exam Consolidation | Next |

---

## 1. Four words first

Lecture 7 opens with four words. Every section below uses at least one.

| Word | Plain meaning | Example |
| :-- | :-- | :-- |
| **Dependency** | Something a class needs to do its job | `ProductController` needs a `ProductService`, so the service is a dependency of the controller. |
| **Bean** | An object that Spring creates and looks after for you | Your `@Service` and `@RestController` classes become beans. |
| **Container** | Where the beans live. Also called the **application context**. Filled when the app starts. | Spring's box of ready-made objects. |
| **Inject** | Hand a dependency over to a class, usually through its constructor | Spring passes the service into the controller. The controller never calls `new`. |

The first three are Spring words, but the idea underneath them is plain Java, and that is what this phase teaches first.

---

## 2. The problem: a class that builds its own collaborators

A **collaborator** is another object a class works with. Here is the obvious way to give a controller its service:

```java
public class ProductController {

    private ProductService productService;

    public ProductController() {
        this.productService = new ProductService();
    }
}
```

Lecture 7: *"It compiles. It runs. It is wrong."*

| Problem | Why |
| :-- | :-- |
| One private copy per controller | Add a second controller and you get a second service, with its own separate list. Products added through one are invisible to the other. |
| Impossible to swap | You cannot hand it a different service (a test version, a database version). The `new` is **welded** into the constructor. |

The same problem sits one level down. Since Phase 3, the service builds its own storage:

```java
public class ProductService {
    private List<Product> products = new ArrayList<>();
    ...
}
```

**Lecture 7's swap request:** *"Your manager wants the products saved in a real database by Friday, same endpoints."* Today, that means rewriting the service (and anything else that builds things for itself).

The smell has a name: **tight coupling** (Phase 3, Section 9: coupling should be low). A class that creates its own collaborator is welded to that exact class.

**Lecture 7's key takeaway: stop creating your collaborators. Ask for them, and let something else decide which one you get.**

- **Analogy: a phone charger.** A phone with the charger permanently soldered on (tight coupling) is useless if the cable breaks. A phone with a standard socket (low coupling) accepts any charger that fits the socket.

---

## 3. Three terms that sound alike

Lecture 7 separates three ideas with a restaurant analogy:

| Term | Question it answers | Meaning | Restaurant |
| :-- | :-- | :-- | :-- |
| **Dependency Inversion** | WHAT to depend on? | A design principle: depend on an **interface**, not on a concrete class. | A phone charger that fits any UK socket, not one particular power station. |
| **Inversion of Control (IoC)** | WHO creates the objects? | The idea: something outside your class (Spring) creates your objects, not your own code. | You order the dish; the kitchen cooks it. You never walk into the kitchen. |
| **Dependency Injection (DI)** | HOW does a class get them? | The technique: the object you need is passed in, usually through your constructor. | The waiter brings the dish to your table. You just receive it. |

In one sentence (Lecture 7): *Spring uses dependency injection to achieve inversion of control, and depending on interfaces makes the parts easy to swap.*

---

## 4. Constructor injection

### 4.1 Ask through the constructor

Instead of building the collaborator, the class **asks for it as a constructor parameter**:

```java
public class ProductController {

    private final ProductService service;

    public ProductController(ProductService service) {
        this.service = service;
    }
}
```

| Line | What it means |
| :-- | :-- |
| `private final ProductService service;` | `final` (Phase 2): set once, never changed, never left `null`. |
| `public ProductController(ProductService service)` | "To build me, you must hand me a `ProductService`." The dependency is visible in the constructor itself. |
| `this.service = service;` | Store what was handed in. Exactly the Phase 1 pattern. |
| *(no `new ProductService()` anywhere)* | The controller no longer decides which service it gets. |

This is **constructor injection**. Nothing about it is special Java: it is an ordinary constructor with an ordinary parameter. What changed is **who calls `new`**.

### 4.2 Who calls `new` now? In plain Java: you, once, at the top

Someone still has to build the objects and plug them together. In plain Java, that happens once, in `main`:

```java
ProductRepository repository = new InMemoryProductRepository();
ProductService service = new ProductService(repository);
ProductController controller = new ProductController(service);
```

Build the bottom layer first, then pass it upward. The one place where all the objects are built and wired together is sometimes called the **composition root**.

**In Spring, the container does this part for you** at startup (Section 7). That is Inversion of Control: the wiring code above disappears from your project, because Spring writes it.

### 4.3 Why constructor injection wins (Lecture 7)

| Benefit | Meaning |
| :-- | :-- |
| **`final`** | Assigned once during construction. Can never be reassigned and never be `null`. |
| **Explicit** | Read the constructor and you know every collaborator the class needs. |
| **Flexible** | In plain Java you can write `new ProductService(anyRepository)` and choose the parts yourself, for example in a test. |
| **Fails fast** | In Spring, a missing dependency stops the application **at startup**, not at the first request. |

Lecture 7, Q3: *"Your service needs five collaborators and the constructor looks awful. Is that injection's fault?"* **No. Five dependencies means the class does too much. The ugliness is the diagnosis** (low cohesion, Phase 3).

### 4.4 The three injection styles in Spring

| Style | Code | Verdict |
| :-- | :-- | :-- |
| **Field injection** | `@Autowired private ProductService service;` | Avoid. Hides the dependency, allows `null`, forbids `final`, and only works when Spring is running. (What Lecture 3 used.) |
| **Setter injection** | `@Autowired` on a setter | Rarely. Only for a genuinely optional dependency. |
| **Constructor injection** | Dependency as a constructor parameter | **Use this.** The Spring team's own recommendation. |

Since Spring 4.3, a class with **exactly one constructor** is wired automatically, so `@Autowired` is **not needed** on it. Lecture 7's practical rule: **one constructor, every dependency `final`, no `@Autowired`.**

Lecture 7, Q1: *"Why can the field be `final` with constructor injection but not with field injection?"* **Because the constructor assigns it during construction. Field injection sets it afterwards, so `final` is impossible.**

---

## 5. The repository: storage behind an interface

### 5.1 What a repository is

A **repository** is a class whose **only job is storage**: it keeps the objects and gives them back when asked. Four simple actions: `findAll`, `findById`, `save`, `delete`.

- **Analogy (Lecture 7): a library desk.** You ask for a book; the librarian fetches it. You do not know or care whether it was on a shelf, in the basement, or in another building.

The plan:

| | Before | After |
| :-- | :-- | :-- |
| `ProductService` | Holds the `List<Product>` **and** the business rules | Holds **only the rules** |
| Storage | Inside the service | In its own class, behind an interface |

### 5.2 Step 1: the contract (an interface)

```java
import java.util.List;
import java.util.Optional;

public interface ProductRepository {
    List<Product> findAll();
    Optional<Product> findById(Long id);
    Product save(Product product);
    void delete(Long id);
}
```

Four method headers. No storage, no code. It says **what** storage can do, not **how** (Phase 5, Section 5).

### 5.3 Step 2: one implementation (the list moves here)

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

public class InMemoryProductRepository implements ProductRepository {

    private final List<Product> products = new ArrayList<>();
    private Long nextId = 1L;

    @Override
    public List<Product> findAll() {
        return products;
    }

    @Override
    public Optional<Product> findById(Long id) {
        return products.stream()
                .filter(p -> p.getId().equals(id))
                .findFirst();
    }

    @Override
    public Product save(Product product) {
        if (product.getId() == null) {
            product.setId(nextId++);
        }
        products.add(product);
        return product;
    }

    @Override
    public void delete(Long id) {
        products.removeIf(p -> p.getId().equals(id));
    }
}
```

| Piece | Meaning |
| :-- | :-- |
| `implements ProductRepository` | Signs the contract: must provide all four methods. |
| `private final List<Product> products` | The `ArrayList` that lived in the service since Phase 3 has moved here. `final` means the variable always refers to the same list (the list's contents can still change). |
| `if (product.getId() == null)` | Only new products (with no id yet) get the next id. `== null` is correct here: we are checking "is there an object?". |
| `@Override` on each method | Checks each matches the interface exactly. |

"In memory" means the data lives in the program's memory (RAM): fast, but lost when the program stops. Lecture 7: *like a whiteboard, wiped clean at the end of the day.* A database is *like a notebook: still there tomorrow.* (How Java talks to a database is not part of this module.)

### 5.4 Step 3: the service asks for the interface

```java
import java.util.List;

public class ProductService {

    private final ProductRepository repository;

    public ProductService(ProductRepository repository) {
        this.repository = repository;
    }

    public List<Product> getAllProducts() {
        return repository.findAll();
    }

    public Product getProductById(Long id) {
        return repository.findById(id)
                .orElseThrow(() -> new ProductNotFoundException(id));
    }

    public Product addProduct(Product product) {
        if (product.getName() == null || product.getName().isBlank()) {
            throw new InvalidProductException("Name is required");
        }
        if (product.getPrice() <= 0) {
            throw new InvalidProductException("Price must be greater than zero");
        }
        return repository.save(product);
    }

    public void deleteProduct(Long id) {
        getProductById(id);
        repository.delete(id);
    }
}
```

| Line | Meaning |
| :-- | :-- |
| `private final ProductRepository repository;` | The type is the **interface**. The service never names `InMemoryProductRepository`. |
| Constructor parameter `ProductRepository repository` | "Hand me any repository." |
| `repository.findById(id).orElseThrow(...)` | The storage finds; the **service** decides that "missing" is an error (Phase 6). |
| `addProduct` | Validation (a business rule) stays in the service; saving is delegated to the repository. |
| `deleteProduct` calls `getProductById(id)` first | Reuses the not-found check: deleting a missing id throws `ProductNotFoundException` (which becomes a 404). |

The service now holds **business rules and nothing else**. That is high cohesion (Phase 3).

Lecture 7, Q1: *"Why does `ProductService` declare `ProductRepository` rather than `InMemoryProductRepository`?"* **So the implementation can be swapped without touching the service. It depends on a contract, not on a class.**

---

## 6. The payoff: swapping without touching anything

### 6.1 A second implementation

Here is a different storage, using a `Map` (Phase 3). In Lecture 7 the example is a database class written by a teammate; the idea is identical.

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;

public class MapProductRepository implements ProductRepository {

    private final Map<Long, Product> products = new HashMap<>();
    private Long nextId = 1L;

    @Override
    public List<Product> findAll() {
        return new ArrayList<>(products.values());
    }

    @Override
    public Optional<Product> findById(Long id) {
        return Optional.ofNullable(products.get(id));
    }

    @Override
    public Product save(Product product) {
        if (product.getId() == null) {
            product.setId(nextId++);
        }
        products.put(product.getId(), product);
        return product;
    }

    @Override
    public void delete(Long id) {
        products.remove(id);
    }
}
```

### 6.2 Swap it in one line

```java
public class LayersDemo {

    public static void main(String[] args) {
        ProductRepository repository = new InMemoryProductRepository();
        // ProductRepository repository = new MapProductRepository();   // the only line to change

        ProductService service = new ProductService(repository);

        service.addProduct(new Product(null, "Laptop", 899.99));
        service.addProduct(new Product(null, "Mouse", 24.99));

        System.out.println(service.getAllProducts().size());
        System.out.println(service.getProductById(2L).getName());

        service.deleteProduct(1L);
        System.out.println(service.getAllProducts().size());

        try {
            service.getProductById(1L);
        } catch (ProductNotFoundException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

**Output (identical with either repository):**

```
2
Mouse
1
Product not found with id: 1
```

Lecture 7's vote, *"how many existing files change?"*:

| File | Changes? | Why |
| :-- | :-- | :-- |
| `ProductService` | No | It asked for the interface, not the class. |
| `ProductController` | No | It never knew a repository existed. |
| The endpoints | No | Same URLs, same JSON, same status codes. |
| **Total** | **Zero existing files** | One new file added; one line of wiring changed (and in Spring, not even that). |

That is the **modularity** your coursework report (LO4) has to evidence, and it is Phase 5's promise collected: Lecture 7 says *an interface is worth writing when somebody other than the caller decides which implementation turns up.*

### 6.3 A bonus: easy testing

Because the service accepts **any** repository, you can test its rules with a fresh, empty one, without starting a web server:

```java
ProductService service = new ProductService(new InMemoryProductRepository());
Product saved = service.addProduct(new Product(null, "Mug", 8.50));
System.out.println(saved.getId());   // 1
```

Lecture 8 (JUnit, Mockito) builds on exactly this. Field injection makes it impossible: there is no way to hand the class a different object.

---

## 7. How Spring does the wiring

### 7.1 The four labels (Lecture 7)

| Annotation | Goes on | Meaning |
| :-- | :-- | :-- |
| `@Component` | Any class, when nothing more specific fits | "Spring, make an object of this class." |
| `@Service` | `ProductService` | Same as `@Component`; the name tells readers "business rules live here". |
| `@Repository` | `InMemoryProductRepository` (the class) | "This class stores data." **Never on the interface.** |
| `@RestController` | `ProductController` | "This class answers web requests; return values become JSON." |

```java
@RestController
public class ProductController { ... }

@Service
public class ProductService { ... }

@Repository
public class InMemoryProductRepository implements ProductRepository { ... }

public interface ProductRepository { ... }     // NO label: Spring cannot make an object of an interface
```

### 7.2 What Spring does when you press Run (Lecture 7)

| Step | What happens |
| :-- | :-- |
| 1. Component scan | Spring searches every package **below** your `@SpringBootApplication` class for those labels. |
| 2. Register | Each labelled class becomes a **bean definition** in the container. |
| 3. Resolve | For each bean, Spring reads its constructor parameters and finds a bean of the matching **type**. `ProductService` asks for a `ProductRepository`; the only bean implementing it is `InMemoryProductRepository`, so that one is passed in. |
| 4. Ready | Objects are created bottom-up and wired together. Only then does the web server accept requests. |

That is exactly the three lines from Section 4.2, written by Spring instead of you. **You never call `new` on a bean.**

Lecture 7 quick checks:

| Question | Answer |
| :-- | :-- |
| Who creates the `ProductService` in a Spring Boot app? | The application context, during startup. |
| What exactly is being inverted in IoC? | Control over object creation and lifetime. It moves out of your classes and into the container. |
| Your class has no annotation, or sits outside the scanned packages. Will Spring inject it? | No. No label, or the wrong package, means no bean. |

### 7.3 One shared object: singleton scope

By default Spring makes **one** object of each labelled class, and every class that asks for it gets **that same object**. This is called **singleton scope** ("scope" = how many objects Spring makes).

That is why a product you POST is still there on the next GET: there is one repository, shared.

**Lecture 7's warning: never keep one user's data in a field of a service.**

```java
// WRONG
@Service
public class CartService {
    private List<Product> cart = new ArrayList<>();   // ONE cart, shared by EVERYONE
}
```

Alice adds an e-book; Bob adds a pen; Alice opens her cart and finds an e-book **and** a pen. There is one `CartService`, so there is one cart for everybody. (The product list in the repository is fine: it is meant to be shared.)

### 7.4 Startup errors you might see

| Message | Cause | Fix |
| :-- | :-- | :-- |
| *"Parameter 0 of constructor in ProductService required a bean of type 'ProductRepository' that could not be found"* / *"Consider defining a bean of type ProductRepository"* | The implementation is missing `@Repository`, or is outside the scanned package. | Add the label; move the file under the main application's package. |
| *"required a single bean, but 2 were found"* | Two classes implement `ProductRepository`; Spring cannot choose. | Delete the one you do not need, or put `@Primary` on the one to use. |

---

## 8. The layers

```
 +-------------------------------------------------------------+
 |  @RestController   ProductController                        |
 |  Speaks HTTP: URLs, JSON, success status codes.             |
 |  No business rules.                                          |
 +-------------------------------------------------------------+
                         |  depends on
                         v
 +-------------------------------------------------------------+
 |  @Service          ProductService                           |
 |  Business rules: validates, calculates, throws exceptions.  |
 |  Knows nothing about HTTP.                                   |
 +-------------------------------------------------------------+
                         |  depends on the INTERFACE
                         v
 +-------------------------------------------------------------+
 |  «interface»       ProductRepository                        |
 +-------------------------------------------------------------+
                        /_\   (implements)
                         :
 +-------------------------------------------------------------+
 |  @Repository       InMemoryProductRepository                |
 |  Storage: an ArrayList today, a database tomorrow.          |
 |  Knows nothing about business rules.                        |
 +-------------------------------------------------------------+

 Beside them:  @ControllerAdvice  GlobalExceptionHandler
               turns the service's exceptions into 404 / 400.
```

Lecture 7's rule: **each layer only talks to the layer directly below it, and the service reaches storage through an interface.** Every arrow points down; none points back up.

### 8.1 Which layer? (Lecture 7's activity)

| Responsibility | Layer |
| :-- | :-- |
| Converting a `Product` to JSON | `@RestController` (Jackson does it at the web boundary) |
| Rejecting a negative price | `@Service` (a business rule) |
| Generating the next id | `@Repository` (storage owns identity) |
| Turning a not-found exception into 404 | `@ControllerAdvice` |
| Applying a 20% discount cap | `@Service` (or the model class itself, as in Phase 5) |
| Choosing `ArrayList` or database | `@Repository` (the only class that should know) |

### 8.2 Patterns you have now built (Lecture 7, for your report)

| Pattern | Where |
| :-- | :-- |
| **Repository** | Storage behind a contract: `ProductRepository`. |
| **Strategy** | Interchangeable implementations of one contract chosen at runtime: `Discountable` (Phase 5), and the repositories here. |
| **Layered architecture** | Controller → Service → Repository, each with one job. |
| **Dependency injection** | Collaborators supplied through constructors, never created. |

### 8.3 Reading an exam starter in five minutes

When the timer starts, open the files in this order and answer each question:

1. **The abstract parent** (`model/`): which fields and constructor parameters (in which order)? Which abstract methods must every subclass override? Are the fields `private` (use getters) or `protected`?
2. **The existing subclass**: what does a finished-looking one have? Copy its shape.
3. **The repository interface**: what are the method names (`findById`? `save`?) and what do they return (`Optional`?)? Your service will call these.
4. **The service**: what is already injected through the constructor? Where are the `// TODO`s?
5. **`ErrorResponse`**: what does its constructor take?
6. **The controller**: which endpoints exist, and which are `// TODO`?
7. **The exception package**: which exceptions exist already?

---

## 9. Common mistakes

| Mistake | Problem | Fix |
| :-- | :-- | :-- |
| `new ProductService()` inside the controller | Tight coupling; a second, separate service. | Ask for it through the constructor. |
| Declaring the field as the class, not the interface (`InMemoryProductRepository repo`) | Cannot swap. | Use the interface type. |
| `@Repository` on the interface | Spring cannot make an object of an interface. | Put it on the implementing class. |
| Forgetting a label | "Consider defining a bean of type...". | Add `@Service` / `@Repository`. |
| Two implementations with no `@Primary` | "required a single bean, but 2 were found". | Remove one or mark one `@Primary`. |
| Per-user data in a service field | Shared by every user. | Keep it out of singletons. |
| Business rules in the repository | Storage and rules tangled. | Repository stores; service decides. |
| Editing the provided repository in the exam | Explicitly not allowed. | Read it; call its methods from the service. |

---

## 10. Practice

### 10.1 Quick questions

| # | Question | Answer |
| :-- | :-- | :-- |
| 1 | What is a dependency? | Something a class needs to do its job. |
| 2 | What is dependency injection? | Passing a class its dependencies (usually through the constructor) instead of letting it create them. |
| 3 | What is inverted in Inversion of Control? | Control over creating objects: it moves from your classes to the container. |
| 4 | What does Dependency Inversion say? | Depend on interfaces (abstractions), not concrete classes. |
| 5 | Why is constructor injection preferred? | `final` fields, explicit dependencies, usable without Spring, fails at startup. |
| 6 | Which layer should the `List<Product>` live in? | The repository. Storage is its single responsibility. |
| 7 | Where does `@Repository` go? | On the implementing class, never the interface. |
| 8 | What is singleton scope, and its danger? | One shared object per class; never store per-user data in its fields. |
| 9 | You swap to a database repository. How many existing files change? | None (apart from the wiring, which Spring does). |

### 10.2 Build it: a library with a swappable repository

Using the `Book` class from Phase 3:

1. Write a `BookRepository` interface: `findAll()`, `findById(Long)` returning `Optional<Book>`, `save(Book)`.
2. Write `InMemoryBookRepository` implementing it with a `List` and an id counter.
3. Write `BookNotFoundException` (Phase 6 style).
4. Write `LibraryService` that receives a `BookRepository` through its constructor (field `final`), with `addBook`, `findById` (throws if missing) and `countAvailable`.
5. In `main`, wire them together and test.

Try for 25 minutes.

```java
import java.util.List;
import java.util.Optional;

public interface BookRepository {
    List<Book> findAll();
    Optional<Book> findById(Long id);
    Book save(Book book);
}
```

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

public class InMemoryBookRepository implements BookRepository {

    private final List<Book> books = new ArrayList<>();
    private Long nextId = 1L;

    @Override
    public List<Book> findAll() {
        return books;
    }

    @Override
    public Optional<Book> findById(Long id) {
        return books.stream()
                .filter(b -> b.getId().equals(id))
                .findFirst();
    }

    @Override
    public Book save(Book book) {
        if (book.getId() == null) {
            book.setId(nextId++);
        }
        books.add(book);
        return book;
    }
}
```

```java
public class BookNotFoundException extends RuntimeException {

    public BookNotFoundException(Long id) {
        super("Book not found with id: " + id);
    }
}
```

```java
public class LibraryService {

    private final BookRepository repository;

    public LibraryService(BookRepository repository) {
        this.repository = repository;
    }

    public Book addBook(Book book) {
        return repository.save(book);
    }

    public Book findById(Long id) {
        return repository.findById(id)
                .orElseThrow(() -> new BookNotFoundException(id));
    }

    public long countAvailable() {
        return repository.findAll().stream()
                .filter(b -> !b.isOnLoan())
                .count();
    }
}
```

```java
BookRepository repository = new InMemoryBookRepository();
LibraryService library = new LibraryService(repository);

library.addBook(new Book(null, "Dune", "Frank Herbert"));
Book emma = library.addBook(new Book(null, "Emma", "Jane Austen"));
emma.setOnLoan(true);

System.out.println(library.findById(1L).getTitle());   // Dune
System.out.println(library.countAvailable());          // 1

try {
    library.findById(42L);
} catch (BookNotFoundException e) {
    System.out.println(e.getMessage());                // Book not found with id: 42
}
```

---

## 11. Glossary

| Term | Plain-English meaning |
| :-- | :-- |
| **Application context** | Spring's container of beans. |
| **Bean** | An object Spring creates and manages. |
| **Collaborator** | Another object a class works with. |
| **`@Component`** | General Spring label: make a bean of this class. |
| **Component scan** | Spring searching packages below the main class for labels. |
| **Composition root** | The one place where all objects are built and wired together. |
| **Constructor injection** | Receiving dependencies as constructor parameters. |
| **Container** | Where beans live; builds and wires them at startup. |
| **Dependency** | Something a class needs to do its job. |
| **Dependency Injection (DI)** | Handing a class its dependencies instead of it creating them. |
| **Dependency Inversion** | Depend on interfaces, not concrete classes. |
| **Field injection** | `@Autowired` on a field. Avoid. |
| **In-memory** | Stored in the running program's memory; lost on restart. |
| **Inversion of Control (IoC)** | The framework, not your code, creates objects and calls you. |
| **Layered architecture** | Controller → service → repository, each with one job. |
| **`@Primary`** | Marks which bean to use when several match. |
| **Repository** | A class whose only job is storage. |
| **`@Repository`** | Spring label for a storage class. |
| **`@RestController`** | Spring label for a class that answers web requests. |
| **`@Service`** | Spring label for a business-rules class. |
| **Singleton scope** | One shared bean object per class (Spring's default). |
| **Strategy pattern** | Interchangeable implementations of one interface. |
| **Tight coupling** | A class welded to another, typically by creating it with `new`. |

---

## 12. One-page cheat sheet

```java
// ---- the contract ----
public interface ThingRepository {                 // NO annotation on the interface
    List<Thing> findAll();
    Optional<Thing> findById(Long id);
    Thing save(Thing t);
    void delete(Long id);
}

// ---- storage ----
@Repository
public class InMemoryThingRepository implements ThingRepository {
    private final List<Thing> things = new ArrayList<>();
    private Long nextId = 1L;
    @Override public Thing save(Thing t) {
        if (t.getId() == null) t.setId(nextId++);
        things.add(t);
        return t;
    }
    // findAll, findById, delete ...
}

// ---- rules ----
@Service
public class ThingService {
    private final ThingRepository repository;          // interface type, final
    public ThingService(ThingRepository repository) {   // constructor injection, no @Autowired
        this.repository = repository;
    }
    public Thing findById(Long id) {
        return repository.findById(id).orElseThrow(() -> new ThingNotFoundException(id));
    }
}

// ---- web ----
@RestController
public class ThingController {
    private final ThingService service;
    public ThingController(ThingService service) { this.service = service; }
}

// ---- plain-Java wiring (Spring does this for you) ----
ThingRepository repo = new InMemoryThingRepository();
ThingService service = new ThingService(repo);
ThingController controller = new ThingController(service);
```

| Remember | |
| :-- | :-- |
| Do not `new` your collaborators | Ask for them in the constructor |
| Dependency fields are `final` | One constructor, no `@Autowired` |
| Depend on the interface | Swap implementations without editing callers |
| Repository stores; service decides; controller speaks HTTP | Arrows only point down |
| `@Repository` on the class, not the interface | Missing label = "no bean of type" |
| One bean per class, shared by all | No per-user data in service fields |
| In the exam: read the provided structure first | Do not edit the repository or parent |

---

## 13. Am I ready for Phase 8?

- [ ] I can explain dependency, bean, container and inject. (§1)
- [ ] I can explain why `new ProductService()` inside a controller is a problem. (§2)
- [ ] I can tell Dependency Inversion, IoC and DI apart. (§3)
- [ ] I can write constructor injection with a `final` field. (§4)
- [ ] I can write a repository interface and an in-memory implementation. (§5)
- [ ] I can write a service that depends on the repository interface. (§5.4)
- [ ] I can show that swapping implementations changes no other file. (§6)
- [ ] I know which label goes on which class, and why the interface has none. (§7.1)
- [ ] I can explain singleton scope and its danger. (§7.3)
- [ ] I can draw the three layers and say what belongs in each. (§8)
- [ ] I know the order to read an exam starter project. (§8.3)
- [ ] I built the library with a swappable repository without copying. (§10.2)

**Next: Phase 8, Exam Consolidation.** Everything from Phases 0 to 7, assembled into one timed run of the exam's shape.
