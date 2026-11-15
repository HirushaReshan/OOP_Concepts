# S3: The Service Layer, and Inheritance and Polymorphism in a Spring API

**Before this:** S2 (endpoints, Jackson, records). OOP Phases 3, 4 and 5 (collections, inheritance, polymorphism) are assumed; this guide shows how they live inside a Spring project.
**Goal:** by the end you can split a controller from a service, write a full CRUD API through a service, put a class hierarchy behind one endpoint, make each subclass answer differently through one service loop, and recognise exactly this pattern in the practice papers.
**Time needed:** about 2 hours.

**Built from your files:** *Introduction to Service Layer and Separating Controllers from Services* (25 pages), *Inheritance in Spring Boot: Refactoring Code Duplication into Class Hierarchies* (24 pages), *Polymorphism in Spring Boot: Method Overriding, Abstract Classes and Interfaces* (18 pages), and the model and service files of your three practice starters.

> **Exam link.** Tasks 1 and 2 (40 marks) are "complete one subclass, create another, and process them polymorphically in the service". Task 3 is service methods. Every practice paper says *"Do not use instanceof or branch on concrete types"*. This guide is the Spring side of OOP Phases 4 and 5.

---

## Part A: The service layer

## 1. Controller's job versus service's job

| Controller layer (the **web** layer) | Service layer (the **business** layer) |
| :-- | :-- |
| Receives HTTP requests, returns HTTP responses | Runs the business logic: assigning ids, finding, calculating, validating, saving, deleting |
| Reads path variables, query parameters and request bodies | Knows **nothing** about HTTP: no URLs, no status codes |
| Knows URLs and status codes, and **no business rules** | Can be tested on its own, with no server running |

Lecture: *if business logic lives in the controller, the controller becomes untestable and unmaintainable.*

- **Analogy: a restaurant.** The waiter (controller) takes orders and brings plates; the chef (service) decides how to cook. A waiter who also cooks is slow and messy, and you cannot test the recipe without a customer.

### 1.1 Where does it belong? (lecture activity)

| Line of code | Layer | Why |
| :-- | :-- | :-- |
| Reject a product whose price is below zero | Service | A business rule |
| Read the id out of the URL with `@PathVariable` | Controller | Pure HTTP |
| Give a new product the next available id | Service | The rule owns the ids |
| Return 404 when nothing matches | Controller side (exception handler, S4) | 404 is an HTTP status |
| Filter the catalogue to products under £50 | Service | Filtering is logic |

### 1.2 The journey of one request (`POST /products`)

1. **The request arrives at the controller.** Spring matches URL and method and turns the JSON into a `Product`.
2. **The controller delegates immediately:** one line, `productService.addProduct(product)`.
3. **The service applies the rules:** assigns the next id, stores the product.
4. **The service returns the saved object:** a plain Java object; the service has no idea a browser exists.
5. **The controller shapes the response:** Spring turns it into JSON and sets the status.

**Rule: requests travel down, answers travel back up. The service never calls the controller.**

---

## 2. Building `ProductService`

### 2.1 The class

```java
@Service
public class ProductService {

    private List<Product> products = new ArrayList<>();
    private Long nextId = 1L;

    public List<Product> getAllProducts() {
        return products;
    }

    public Optional<Product> getProductById(Long id) {
        return products.stream()
                .filter(p -> p.getId().equals(id))
                .findFirst();
    }

    public Product addProduct(Product product) {
        product.setId(nextId);
        nextId += 1L;
        products.add(product);
        return product;
    }

    public String deleteProduct(Long id) {
        boolean isDeleted = products.removeIf(p -> p.getId().equals(id));
        if (isDeleted) {
            return "Deleted successfully";
        } else {
            return "ID not found";
        }
    }

    public List<Product> findProductsByName(String name) {
        return products.stream()
                .filter(p -> p.getName().equalsIgnoreCase(name))
                .toList();
    }
}
```

| Piece | Meaning |
| :-- | :-- |
| `@Service` | "Spring, create **one** object of this class at startup and look after it." It is a business-logic component. Import: `org.springframework.stereotype.Service`. |
| `private List<Product> products` | The data, hidden from the controller. |
| `Optional<Product>` | "A product, or nothing" (OOP Phase 3, §7). |
| `.stream().filter(...).findFirst()` | Scan the list, keep matches, return the first if any. |
| `removeIf(...)` returns `boolean` | `true` if anything was removed. |
| `equalsIgnoreCase` | Compare text ignoring capitals. |

### 2.2 The controller delegates

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping
    public List<Product> getAllProducts() {
        return productService.getAllProducts();
    }

    @GetMapping("/{id}")
    public Product getProductById(@PathVariable Long id) {
        return productService.getProductById(id).orElse(null);   // placeholder; S4 makes it a 404
    }

    @GetMapping("/search")
    public List<Product> search(@RequestParam String name) {
        return productService.findProductsByName(name);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Product saveProduct(@RequestBody Product product) {
        return productService.addProduct(product);
    }

    @DeleteMapping("/{id}")
    public String deleteProductById(@PathVariable Long id) {
        return productService.deleteProduct(id);
    }
}
```

| Endpoint | Annotations | Delegates to |
| :-- | :-- | :-- |
| `GET /products` | `@GetMapping` | `getAllProducts()` |
| `GET /products/{id}` | `@GetMapping` + `@PathVariable` | `getProductById(id)` |
| `GET /products/search?name=` | `@GetMapping` + `@RequestParam` | `findProductsByName(name)` |
| `POST /products` | `@PostMapping` + `@RequestBody` | `addProduct(product)` |
| `DELETE /products/{id}` | `@DeleteMapping` + `@PathVariable` | `deleteProduct(id)` |

> **Slide correction.** On the lecture's "Wiring the Controller" slide, `deleteProductById` is declared to return `String` but its body is only `productService.deleteProduct(id);` with no `return`. That does not compile ("missing return statement"). Add `return`, as above.

### 2.3 `@Autowired`, at surface level

```java
@Autowired
private ProductService productService;
```

Nobody writes `new ProductService()`. At startup Spring creates **one** `ProductService` and **plugs it into** this field because of `@Autowired`. That is enough to make it work.

Your practice starters use the better form, **constructor injection**, which S5 explains:

```java
private final ParkingService service;

public ParkingController(ParkingService service) {
    this.service = service;
}
```

Spring calls this constructor and hands in the service. No `@Autowired` is needed on a class with a single constructor. **In the exam it is already written: read it, do not change it.**

---

## 3. Cohesion and coupling (lecture recap)

| Word | Good means | Test |
| :-- | :-- | :-- |
| **Cohesion** | **High**: a class does one job well (Single Responsibility Principle). `ProductService` manages products only. | One sentence, no "and". |
| **Coupling** | **Low**: few dependencies, one direction only. Controller → service; the service has no idea the controller exists. | Change one class: how many others must change? |

Lecture quick checks: *Does `ProductService` need to know `ProductController` exists?* **No**, and that is what makes the coupling good. *Why can the controller and service be written separately and still work?* **Low coupling: the controller only needs the service's public methods.**

---

## Part B: Inheritance in a Spring API

## 4. The problem: copy-paste models

Adding `DigitalProduct` by copying `Product` duplicates `id`, `name`, `price` and six accessors. It compiles and works, which is why it survives. The lecture lists the dangers:

| Danger | Meaning |
| :-- | :-- |
| Cascading edits | One change must be made in several files. |
| Silent drift | Forget one file and the classes quietly disagree. |
| Doubled testing | Every test exists twice. |

**WET** ("Write Everything Twice") is the anti-pattern; **DRY** ("Don't Repeat Yourself") the principle. Inheritance fixes it (OOP Phase 4).

## 5. The hierarchy

```java
public class Product {

    private Long id;
    private String name;
    private double price;

    public Product() {}                                   // for Jackson

    public Product(Long id, String name, double price) {  // for your code and super(...)
        this.id = id;
        this.name = name;
        this.price = price;
    }

    // getters and setters for id, name, price
}
```

```java
public class DigitalProduct extends Product {

    private String downloadUrl;
    private double fileSizeMB;

    public DigitalProduct() {}                            // for Jackson

    public DigitalProduct(Long id, String name, double price,
                          String downloadUrl, double fileSizeMB) {
        super(id, name, price);                           // must be the first line
        this.downloadUrl = downloadUrl;
        this.fileSizeMB = fileSizeMB;
    }

    public String getDownloadUrl() { return downloadUrl; }
    public void setDownloadUrl(String downloadUrl) { this.downloadUrl = downloadUrl; }

    public double getFileSizeMB() { return fileSizeMB; }
    public void setFileSizeMB(double fileSizeMB) { this.fileSizeMB = fileSizeMB; }
}
```

`PhysicalProduct` is the same shape with `weightKg` and `dimensions`.

> **Slide corrections.** (1) The lecture's `DigitalProduct` slide names a setter `setGetFileSizeMB`. Jackson would treat that as a property called `getFileSizeMB`, so `"fileSizeMB"` in incoming JSON would be ignored. The correct name is `setFileSizeMB`. (2) "One List, Two Specific Product Types" writes `Private Long nextId` with a capital P; Java keywords are lower case: `private`.

### 5.1 One list, two types, one endpoint

```java
@Service
public class ProductService {

    private List<Product> products = new ArrayList<>();
    private Long nextId = 1L;

    public ProductService() {
        products.add(new DigitalProduct(nextId++, "E-book", 9.99, "/dl/1", 4.2));
        products.add(new PhysicalProduct(nextId++, "Monitor", 249.99, 5.4, "60x40"));
    }
    ...
}
```

`List<Product>` holds both because each IS-A `Product`. `GET /products` now returns one array whose objects have **different shapes**:

```json
[
  {"id":1,"name":"E-book","price":9.99,"downloadUrl":"/dl/1","fileSizeMB":4.2},
  {"id":2,"name":"Monitor","price":249.99,"weightKg":5.4,"dimensions":"60x40"}
]
```

Jackson calls the getters of the **real** object (runtime type), inherited getters included. That is why the inherited `id`, `name`, `price` appear without being written twice.

This is exactly what your practice starters' `GET /api/passes` does: an `HourlyPass` shows `rateCents`, a `WeekendPass` shows `flatCents`, and both show `id` and `registration` from the parent.

### 5.2 POSTing a subclass: Jackson polymorphic deserialisation

`@RequestBody Product product` builds a plain `Product`; the extra `downloadUrl` key is ignored. To let Jackson pick the subclass from the JSON (lecture slide):

```java
import com.fasterxml.jackson.annotation.JsonSubTypes;
import com.fasterxml.jackson.annotation.JsonTypeInfo;

@JsonTypeInfo(use = JsonTypeInfo.Id.DEDUCTION)
@JsonSubTypes({
        @JsonSubTypes.Type(DigitalProduct.class),
        @JsonSubTypes.Type(PhysicalProduct.class)
})
public class Product { ... }
```

| Piece | Meaning |
| :-- | :-- |
| `@JsonTypeInfo(use = JsonTypeInfo.Id.DEDUCTION)` | "Work out the subclass from which fields are present." JSON with `downloadUrl` → `DigitalProduct`; with `weightKg` → `PhysicalProduct`. |
| `@JsonSubTypes({...})` | The list of subclasses Jackson is allowed to create. |

The practice papers avoid this problem differently: they POST a **specific** DTO (`POST /api/passes/hourly` with a `NewHourlyPass` record), and the **service** builds the concrete class. That is simpler and is the pattern to use in the exam.

---

## Part C: Polymorphism in a Spring API

## 6. Each type answers differently

Client requirement (lecture): every product type must produce its own summary line, and the UI wants all of them from one call, without knowing the types.

```java
// Product
public String getSummary() {
    return "Name: " + name + " | Price: " + price;
}

// DigitalProduct
@Override
public String getSummary() {
    return "Digital: Name: " + getName() + " | Price: " + getPrice()
            + " | Url: " + downloadUrl + " | FileSize: " + fileSizeMB;
}

// PhysicalProduct
@Override
public String getSummary() {
    return "Physical: Name: " + getName() + " | Price: " + getPrice()
            + " | Weight: " + weightKg + " | Dimensions: " + dimensions;
}
```

`getName()` not `name`: the field is private in the parent (OOP Phase 4).

**Service, one loop, no type checks:**

```java
public List<String> getAllSummaries() {
    return products.stream()
            .map(p -> p.getSummary())
            .toList();
}
```

**Controller:**

```java
@GetMapping("/summaries")
public List<String> getAllProductSummaries() {
    return productService.getAllSummaries();
}
```

`GET /products/summaries`:

```json
[
  "Digital: Name: E-book | Price: 9.99 | Url: /dl/1 | FileSize: 4.2",
  "Physical: Name: Monitor | Price: 249.99 | Weight: 5.4 | Dimensions: 60x40"
]
```

Add a `BundleProduct` tomorrow with its own `getSummary()`, and **this endpoint does not change**. That is the Open/Closed Principle (OOP Phase 5).

### 6.1 Predict (lecture)

```java
Product a = new Product(1L, "Mug", 8.50);
Product b = new DigitalProduct(2L, "E-book", 9.99, "/dl/2", 4.2);
Product c = new PhysicalProduct(3L, "Monitor", 249.99, 5.4, "60*40");
```

`a.getSummary()` → `Name: Mug | Price: 8.5`; `b` → the Digital line; `c` → the Physical line. Same variable type, same call, three answers: **the object decides.**

| Lecture quick check | Answer |
| :-- | :-- |
| `Product p = new DigitalProduct(...)`: which `getSummary()` runs? | `DigitalProduct`'s. |
| Can you call `p.getDownloadUrl()`? | No: the reference type `Product` has no such method. |
| `PhysicalProduct` has no `getSummary()`: what prints? | `Product`'s version; the JVM walks up the hierarchy. |

**JSON side effect:** `getSummary()` is a public getter, so every product's JSON now also contains a `"summary"` field. Harmless, and sometimes useful.

## 7. Abstract parent and interfaces in Spring

```java
public abstract class Product {
    private Long id;
    private String name;
    private double price;

    public Product() {}
    // full constructor, getters, setters

    public abstract String getSummary();
}
```

- `new Product(...)` no longer compiles ("Product is abstract; cannot be instantiated").
- Every concrete subclass must implement `getSummary()`.
- `List<Product>`, parameters and return types still use `Product`.
- **For POST:** Jackson cannot create an abstract class, so `@RequestBody Product` now needs the `@JsonTypeInfo` setup of §5.2, or a concrete type (`@RequestBody DigitalProduct`), or a DTO.

Interfaces add capabilities (lecture's `Discountable`), implemented differently per class: `DigitalProduct` takes the full rate; `PhysicalProduct` caps it at 20% with `Math.min(rate, 0.20)`. See OOP Phase 5, §5.

---

## Part D: How the practice papers use all this

## 8. The pattern in every starter

All three starters have the same skeleton. The parking one:

```java
public abstract class ParkingPass {
    private final long id;
    private final String registration;

    protected ParkingPass(long id, String registration) {
        this.id = id;
        this.registration = registration;
    }

    public long getId() { return id; }
    public String getRegistration() { return registration; }

    public abstract int priceCents(int stayMinutes);
}
```

| Feature | What it means for you |
| :-- | :-- |
| `abstract class` | You cannot `new ParkingPass(...)`. You work with subclasses. |
| `private final` fields | Set once in the constructor; in your subclass use `getId()`, `getRegistration()`. |
| `protected` constructor | Only subclasses can call it, through `super(id, registration)`. |
| **No no-arg constructor, no setters** | These models are immutable. They are never built by Jackson from JSON: the service builds them from a DTO record. So **do not** add a no-arg constructor to your subclass (it could not compile: `final` fields must be set). |
| `public abstract int priceCents(int stayMinutes)` | Every subclass supplies its own rule. Your `@Override` must match **exactly**: `public int priceCents(int stayMinutes)`. |

The dispatch parent has **two** abstract methods (`energyUnits()`, `priority()`), the scheduler parent two as well (`urgencyAt(int)`, `latestStartTick()`). A subclass must override **every one**, or it will not compile.

### 8.1 Polymorphic processing in the service

The paper's rule: *"Invoke priceCents through ParkingPass references... Do not use instanceof or branch on concrete pass types."*

```java
public List<Quote> allQuotes(int minutes) {
    return repo.all().stream()                       // Collection<ParkingPass>
            .map(pass -> toQuote(pass, minutes))     // pass.priceCents(...) inside: polymorphic
            .sorted(Comparator.comparingInt(Quote::priceCents)
                    .thenComparingLong(Quote::id))
            .toList();
}
```

The variable `pass` has type `ParkingPass`; each object runs its own `priceCents`. (S6 explains `Comparator` line by line.)

**Need a type name for the output** (the `passType` field of `Quote`, the `type` of `PlanLine`)? Use:

```java
pass.getClass().getSimpleName()      // "HourlyPass", "WeekendPass"
```

`getClass()` returns the object's real class; `getSimpleName()` its name without the package. It labels the type without branching on it, so it does not break the "no instanceof" rule.

---

## 9. Practical: build the Products API through a service

1. In a Spring Boot project (S1), create `Product` with private fields, a no-arg and a full constructor, getters and setters.
2. Create `ProductService` with `@Service` (§2.1).
3. Create `ProductController` with `@RestController`, `@RequestMapping("/products")` and the five endpoints (§2.2).
4. Restart. Test with Postman or a `.http` file (S2 §11):
   - `POST /products` with `{"name":"Keyboard","price":59.99}` → 201, id 1.
   - `GET /products` → the array contains it.
   - `GET /products/1`, then `DELETE /products/1`, then `GET /products` → gone.
5. Add `DigitalProduct` and `PhysicalProduct` (§5). Seed one of each in the service constructor. Restart; `GET /products` shows two shapes.
6. Add `getSummary()` overrides and `GET /products/summaries` (§6). Restart; every line differs.
7. Make `Product` abstract with an abstract `getSummary()` (§7). Read the compile errors, fix them.

---

## 10. Practice questions

| # | Question | Answer |
| :-- | :-- | :-- |
| 1 | Name two things a controller must not do. | Contain business rules; manage the data list. |
| 2 | What does `@Service` do? | Marks a business-logic class; Spring creates one object and manages it. |
| 3 | Why does `getProductById` return `Optional<Product>`? | It represents "might not be found" without a risky `null`. |
| 4 | High cohesion means? | One class, one job. |
| 5 | Why can one `List<Product>` hold both subclasses? | Each IS-A `Product`. |
| 6 | Why does `GET /products` show `downloadUrl` for one item only? | Jackson uses the getters of each object's real class. |
| 7 | How do you get a summary line per type without `instanceof`? | Override one method in each subclass and call it through the parent type. |
| 8 | Paper says "do not branch on concrete types". Is `getClass().getSimpleName()` allowed for a label? | Yes: it labels, it does not choose behaviour. |
| 9 | Why do the starter models have no no-arg constructor? | They are immutable and built by the service, not by Jackson. |
| 10 | Your subclass compiles with an error "is not abstract and does not override abstract method energyUnits()". Fix? | Implement every abstract method of the parent, with `@Override` and the exact signature. |

---

## 11. Cheat sheet

```java
@Service
public class ThingService {
    private final List<Thing> things = new ArrayList<>();     // or a repository (S5)

    public List<String> summaries() {
        return things.stream().map(Thing::summary).toList();  // polymorphic, no instanceof
    }
}

@RestController
@RequestMapping("/things")
public class ThingController {
    private final ThingService service;
    public ThingController(ThingService service) { this.service = service; }   // constructor injection

    @GetMapping("/summaries")
    public List<String> summaries() { return service.summaries(); }           // one line: delegate
}

public abstract class Thing {
    private final long id;
    protected Thing(long id) { this.id = id; }
    public long getId() { return id; }
    public abstract String summary();
}

public class Small extends Thing {
    public Small(long id) { super(id); }
    @Override public String summary() { return "Small " + getId(); }
}
```

| Remember | |
| :-- | :-- |
| Controller: HTTP only, one line per method | Service: rules and data, no HTTP |
| One list of the parent type holds every subclass | Jackson shows each object's own getters |
| Override in each subclass; call through the parent | Never `instanceof` chains |
| `getClass().getSimpleName()` for a type label | Not a branch |
| Exam models: abstract, `final` fields, protected constructor | Use getters; no no-arg constructor |

**Next: S4, error handling and HTTP status codes in Spring.**
