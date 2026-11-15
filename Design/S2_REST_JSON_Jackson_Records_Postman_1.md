# S2: REST, JSON, Jackson, Records and Testing Your API

**Before this:** S1 (you can run a Spring Boot app and write a GET endpoint).
**Goal:** by the end you can design REST URLs, write GET/POST/PUT/DELETE endpoints, read values from the path, the query string and the JSON body, explain exactly how Jackson turns objects into JSON and back, use Java **records** as request/response shapes, and test everything with Postman, curl or IntelliJ's HTTP Client.
**Time needed:** about 2 hours.

**Built from your files:** *Building REST APIs with Spring Boot: Encapsulation, Jackson Serialization and Postman Fundamentals* (25 pages), the `@RequestParam` slide from *Inheritance in Spring Boot*, and the `dto` packages of your three practice starters.

> **Exam link.** Task 4 (20 marks) is entirely this guide: mapping annotations, path variables, query parameters, request bodies, and returning 201. Every practice starter also uses `record` classes for its DTOs, which your lectures do not cover, so Section 9 teaches them from scratch.

---

## 1. Anatomy of an HTTP request and response

### 1.1 The request

```
POST /api/passes/hourly?dryRun=false HTTP/1.1      <- method, path, query string
Host: localhost:8081                                <- headers
Content-Type: application/json

{"id":3,"registration":"CAR-303","rateCents":300}   <- body
```

| Part | Meaning | Spring annotation that reads it |
| :-- | :-- | :-- |
| **Method** (verb) | What to do: GET, POST, PUT, PATCH, DELETE | `@GetMapping`, `@PostMapping`... |
| **Path** | Which thing: `/api/passes/hourly` | `@RequestMapping` + `@GetMapping("...")` |
| **Path variable** | A changing part of the path: the `1` in `/api/passes/1` | `@PathVariable` |
| **Query string** | Extra options after `?`, as `name=value` pairs joined by `&` | `@RequestParam` |
| **Headers** | Extra information, e.g. `Content-Type: application/json` ("the body is JSON") | (rarely needed) |
| **Body** | The data sent, usually JSON | `@RequestBody` |

### 1.2 The response

```
HTTP/1.1 201 Created                     <- status code
Content-Type: application/json           <- headers

{"id":3,"registration":"CAR-303","rateCents":300}   <- body
```

Every response has **three parts: a status code, headers, a body** (lecture). When your method returns an object, Spring builds all three for you: status 200, `Content-Type: application/json`, and the object as JSON.

---

## 2. REST and the HTTP verbs

**REST** is a style for designing web APIs: URLs name **things** (resources), and the HTTP method says **what to do** to them. An **API** (Application Programming Interface) is the set of requests a program accepts.

### 2.1 The verbs (lecture)

| Verb | Spring | Meaning | Request body? | Example |
| :-- | :-- | :-- | :-- | :-- |
| **GET** | `@GetMapping` | Read. Changes nothing. | No | `GET /products/1` |
| **POST** | `@PostMapping` | Create something new. If it already exists, return an error. | Yes | `POST /products` |
| **PUT** | `@PutMapping` | Replace an existing item (lecture: create it if missing). | Yes | `PUT /products/1` |
| **PATCH** | `@PatchMapping` | Change some fields of an existing item (error if missing). | Yes | `PATCH /products/1` |
| **DELETE** | `@DeleteMapping` | Remove an item. | Usually no | `DELETE /products/1` |

*Every request is a verb plus a path. The verb says what to do; the path says what to do it to.*

### 2.2 The golden URL rule: nouns only

| Bad | Good |
| :-- | :-- |
| `GET /getAllProducts` | `GET /products` |
| `POST /createProduct` | `POST /products` |
| `GET /deleteProduct?id=1` | `DELETE /products/1` |

Collections use **plural nouns** (`/products`, `/passes`, `/missions`). The practice papers follow this, with sub-paths for special views: `/api/passes/quotes`, `/api/missions/plan`.

---

## 3. JSON

**JSON** (JavaScript Object Notation) is plain text for structured data. Despite the name it has nothing to do with running JavaScript; it is simply the format every language can read.

```json
{
  "id": 1,
  "name": "Laptop",
  "price": 999.99,
  "inStock": true,
  "tags": ["electronics", "sale"],
  "supplier": { "name": "Acme", "country": "UK" },
  "discount": null
}
```

| Syntax | Meaning |
| :-- | :-- |
| `{ }` | One object. |
| `"key": value` | One field. Keys always in double quotes. Fields separated by commas. |
| `"text"` | A string. |
| `1`, `999.99` | Numbers, no quotes. |
| `true` / `false` | Booleans, no quotes. |
| `[ ... ]` | An array (list), e.g. what a `List<Product>` becomes. |
| Nested `{ }` | An object inside an object. |
| `null` | No value. |

Common JSON typing mistakes: a trailing comma after the last field, single quotes instead of double, a missing closing brace. Any of them makes Spring reply **400 Bad Request** before your code runs.

---

## 4. Jackson: how objects become JSON and back

**Jackson** is the library (included in `spring-boot-starter-web`) that converts between Java objects and JSON.

| Direction | Name | When |
| :-- | :-- | :-- |
| Object → JSON | **Serialisation** | Your method returns an object; Spring writes it into the response. |
| JSON → object | **Deserialisation** | A POST arrives with a JSON body; Spring builds the parameter object. |

### 4.1 Serialisation reads **getters**, not fields

The lecture's "break it on purpose" demo:

```java
public class Product {
    private Long id;
    private String name;
    private double price;

    public Product() {}
    public Product(Long id, String name, double price) { ... }
    // no getters yet...
}
```

`GET /products/1` returns `{ }`. **Empty JSON, status 200, no error anywhere.**

Why: Jackson cannot see `private` fields. It looks for public **getter** methods and makes one JSON field per getter:

| Method in the class | JSON field produced |
| :-- | :-- |
| `public Long getId()` | `"id"` |
| `public String getName()` | `"name"` |
| `public boolean isInStock()` | `"inStock"` (`is` for booleans) |
| `public String getSummary()` | `"summary"`, **even with no field called summary** |
| `public int priceCents(int minutes)` | nothing: not a getter name, and it has a parameter |
| `private String getSecret()` | nothing: not public |

Rules that follow:

- **No getter → the field is silently missing from the JSON.** This is the most common "my API returns the wrong thing" bug.
- **Getter misspelled** (`getname()`) → missing too.
- **Every public `getX()` is a field**, so a method like `getSummary()` shows up in the JSON. Usually harmless, and sometimes handy.

Lecture takeaway: *private fields protect state; public getters choose what the API exposes. Encapsulation directly dictates Spring's JSON output.*

### 4.2 Why public fields "work" but are wrong

With `public Long id;` etc., the JSON looks perfect. The problem is that **any code can change the object's data without the object's knowledge** (OOP Phase 2). So: private fields + public getters.

### 4.3 Deserialisation needs a **no-arg constructor** (for ordinary classes)

For a normal class, Jackson builds an object from JSON in two steps:

1. Call the **no-argument constructor**: `new Product()`.
2. Call each **setter** (or set each field) that matches a JSON key: `setName("Laptop")`, `setPrice(999.99)`.

```java
// Product with ONLY the full constructor
@PostMapping("/products")
public Product saveProduct(@RequestBody Product product) { return product; }
```

`POST /products` with `{"id":1,"name":"Mouse","price":19.99}` → **HTTP 400**, because Jackson cannot construct `Product`. The fix is one line:

```java
public Product() {}
```

**The pattern (lecture): two constructors, two audiences.** The no-arg one for frameworks; the full one for your own code (and for `super(...)` in subclasses).

> **Records are the exception.** A Java `record` (Section 9) has no no-arg constructor, yet Jackson builds it fine: it calls the record's one constructor with the JSON values. That is why the practice papers can use records for request bodies.

### 4.4 JSON names must match exactly

The JSON key `"registration"` fills the property `registration`. A key with a different name (`"reg"`) is ignored by default, so the field is left at its default (`null`, `0`, `false`). **Keys are case-sensitive.**

What a missing key becomes:

| Java type | Missing from JSON → |
| :-- | :-- |
| `String`, `Long`, any object | `null` |
| `int`, `long`, `double` | `0` |
| `boolean` | `false` |

This matters for validation: a POST that leaves out `"id"` gives `id = 0`, which your "id must be positive" check then rejects.

---

## 5. Mapping annotations

### 5.1 Class-level base path: `@RequestMapping`

```java
@RestController
@RequestMapping("/api/passes")
public class ParkingController {

    @GetMapping                       // GET  /api/passes
    public Collection<ParkingPass> list() { ... }

    @GetMapping("/{id}/quote")        // GET  /api/passes/{id}/quote
    public Quote quote(...) { ... }

    @PostMapping("/hourly")           // POST /api/passes/hourly
    public ParkingPass addHourly(...) { ... }
}
```

| Piece | Meaning |
| :-- | :-- |
| `@RequestMapping("/api/passes")` on the class | Every method's path starts with this. |
| `@GetMapping` with no path | Exactly the base path. |
| `@GetMapping("/{id}/quote")` | Base path + this. |

The method-level annotations are shortcuts: `@GetMapping("/x")` is the same as `@RequestMapping(value = "/x", method = RequestMethod.GET)`.

### 5.2 Return types and what the client receives

| Your method returns | Response body | Status |
| :-- | :-- | :-- |
| An object (`Product`, a record) | A JSON object | 200 |
| `List<...>` or `Collection<...>` | A JSON array | 200 |
| `Map<String, Object>` | A JSON object with those keys | 200 |
| `String` | Plain text | 200 |
| `void` | Empty | 200 |
| `ResponseEntity<T>` | Whatever you choose (S4) | Whatever you choose |

Status 201 for POST and 204 for DELETE need `@ResponseStatus` (Section 8) or `ResponseEntity`.

---

## 6. `@PathVariable`: values inside the path

```java
@GetMapping("/products/{id}")
public Product getProductById(@PathVariable Long id) {
    ...
}
```

| Piece | Meaning |
| :-- | :-- |
| `{id}` in the path | A placeholder that matches any one path segment. |
| `@PathVariable Long id` | Copy that segment into the parameter `id`. |
| Type conversion | Spring converts the text `"1"` to the number `1L`. `GET /products/abc` with a `Long` parameter → **400**. |
| Name matching | The parameter name must equal the placeholder name. Otherwise write `@PathVariable("id") Long productId`. |

```
GET /products/1      ->   id = 1L
GET /api/passes/2/quote?minutes=481   ->   id = 2
```

**Use a path variable to identify one specific item** (lecture's "quick distinction").

`long` (primitive) works as well as `Long`; the practice starters use `long id`.

---

## 7. `@RequestParam`: values from the query string

```java
@GetMapping("/search")
public List<Product> getProductsByName(@RequestParam String name) {
    return productService.findProductsByName(name);
}
```

`GET http://localhost:8080/products/search?name=Laptop` → `name = "Laptop"`.

| Feature | Meaning |
| :-- | :-- |
| `?name=Laptop` | The **query string**: `key=value`, several joined with `&`. |
| Required by default | If `?name=` is left out entirely → **400 Bad Request** before your method runs. |
| Type conversion | `@RequestParam int minutes` with `?minutes=61` → `61`. `?minutes=abc` → **400**. |
| Optional with a default | `@RequestParam(defaultValue = "false") boolean urgentOnly` |
| Optional, may be null | `@RequestParam(required = false) Integer limit` (use the wrapper type, since a primitive cannot be `null`) |

Two parameters, as in the dispatch paper:

```java
@GetMapping("/plan")
public DispatchPlan plan(@RequestParam int capacity,
                         @RequestParam(defaultValue = "false") boolean urgentOnly) {
    return service.plan(capacity, urgentOnly);
}
```

`GET /api/missions/plan?capacity=60&urgentOnly=true` → `capacity = 60`, `urgentOnly = true`.
`GET /api/missions/plan?capacity=60` → `urgentOnly = false` (the default).

### 7.1 Path variable or query parameter?

| Use | When | Example |
| :-- | :-- | :-- |
| `@PathVariable` | Identifies **which** item | `/passes/2` |
| `@RequestParam` | An **option** that changes the result: filter, search, size, time | `/passes/2/quote?minutes=61` |

---

## 8. `@RequestBody`: the JSON body

```java
@PostMapping("/products")
@ResponseStatus(HttpStatus.CREATED)
public Product saveProduct(@RequestBody Product product) {
    return productService.addProduct(product);
}
```

| Piece | Meaning |
| :-- | :-- |
| `@RequestBody Product product` | Jackson reads the JSON body and builds a `Product` (Section 4.3). |
| `@ResponseStatus(HttpStatus.CREATED)` | Reply **201 Created** instead of 200. Correct for a POST that creates something. |
| `return ...` | The saved object goes back as JSON, so the client learns its new id. |

Import `HttpStatus` from `org.springframework.http`.

- Missing body, or broken JSON → **400**.
- The class has only a full constructor and is not a record → **400** ("cannot construct").
- `@RequestBody` on a **GET** is unusual; GET requests normally have no body.

---

## 9. Java records: the DTOs in your exam starters

### 9.1 What a DTO is

A **DTO** (Data Transfer Object) is a small class whose only job is to **carry data** in or out of the API: the shape of a request body or a response. It has no business logic.

Why have them instead of returning model classes directly?

| Reason | Example from the papers |
| :-- | :-- |
| The request does not match a model class | `NewHourlyPass(id, registration, rateCents)` is what a client sends; the service then builds an `HourlyPass`. |
| The response is a calculated view, not a stored object | `Quote(id, registration, passType, stayMinutes, priceCents)` |
| A consistent error shape | `ApiError(code, message)` |

### 9.2 The `record` keyword

A **record** is a short way to write an **immutable data class**. One line:

```java
public record Quote(long id, String registration, String passType, int stayMinutes, int priceCents) {}
```

…makes Java generate all of this automatically:

| Generated | What it looks like |
| :-- | :-- |
| `private final` fields | `id`, `registration`, `passType`, `stayMinutes`, `priceCents` |
| One constructor taking every field, in order (the **canonical constructor**) | `new Quote(1, "CAR-101", "HourlyPass", 61, 500)` |
| An **accessor** per field, named **exactly like the field** (no `get`) | `quote.id()`, `quote.priceCents()` |
| `equals()` | Two records are equal if **all** fields are equal |
| `hashCode()` | Matching `equals` |
| `toString()` | `Quote[id=1, registration=CAR-101, passType=HourlyPass, stayMinutes=61, priceCents=500]` |

Line by line: `public record Quote(...) {}`

| Piece | Meaning |
| :-- | :-- |
| `record` | Instead of `class`. |
| `Quote` | The type name. |
| `(long id, String registration, ...)` | The **components**: the fields, their types and their order. |
| `{}` | An optional body. Usually empty. |

### 9.3 The rules that trip people up

| Rule | Consequence |
| :-- | :-- |
| Accessors have **no `get` prefix** | `body.id()`, **not** `body.getId()`. `getId()` is "cannot find symbol". |
| Fields are `final` | No setters. You cannot change a record; you make a new one. |
| A record cannot `extend` another class | (It may `implement` interfaces.) |
| No no-arg constructor | `new Quote()` does not compile. Jackson does not need one for records. |
| `equals` compares values | Very useful: the EXTREME paper checks "same request?" with `stored.equals(incoming)`. |

### 9.4 Records with Jackson

| Direction | How it works |
| :-- | :-- |
| Serialisation | Jackson recognises records and uses the accessors: `Quote` → `{"id":1,"registration":"CAR-101",...}`. The JSON keys are the component names. |
| Deserialisation | Jackson calls the canonical constructor with the JSON values. Missing keys → `null` / `0` / `false` (Section 4.4). |

So `@RequestBody NewHourlyPass body` with `{"id":3,"registration":"CAR-303","rateCents":300}` gives a record where `body.id()` is `3`, `body.registration()` is `"CAR-303"`, `body.rateCents()` is `300`.

### 9.5 A record can have methods

From the EXTREME starter:

```java
public record CommitRequest(String runKey, int nowTick, int horizon, int cpuCapacity, int memoryCapacity) {
    public PlanRequest asPlanRequest() {
        return new PlanRequest(nowTick, horizon, cpuCapacity, memoryCapacity);
    }
}
```

Inside the record, components can be used by name (`nowTick`). `asPlanRequest()` builds a different record from four of its five values.

(Jackson ignores `asPlanRequest()` because the name does not start with `get`/`is`.)

### 9.6 Record or class? (for the exam: use what the starter gives you)

| Use a record when | Use a class when |
| :-- | :-- |
| Pure data in or out (DTOs) | It has behaviour, inheritance, or changing state (models) |
| Immutable is fine | You need setters or a parent class |

The papers say: *"Do not replace the parent class, DTO contracts or repository API."* The DTO records are a **contract**: do not change their components.

---

## 10. Worked example: the lecture's Products API (controller only)

Before services exist (S3), the whole API can live in a controller. This version is complete and runnable.

```java
package uk.ac.westminster.productsapi;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;

@RestController
@RequestMapping("/products")
public class ProductController {

    private final List<Product> products = new ArrayList<>();
    private Long nextId = 1L;

    @GetMapping
    public List<Product> getAll() {
        return products;
    }

    @GetMapping("/{id}")
    public Product getById(@PathVariable Long id) {
        return products.stream()
                .filter(p -> p.getId().equals(id))
                .findFirst()
                .orElse(null);                 // placeholder: S4 turns this into a proper 404
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Product create(@RequestBody Product product) {
        product.setId(nextId++);
        products.add(product);
        return product;
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable Long id) {
        products.removeIf(p -> p.getId().equals(id));
    }
}
```

`import org.springframework.web.bind.annotation.*;` imports every annotation in that package at once (the `*` means "all").

`Product` must have: private fields, **a no-arg constructor**, a full constructor, getters, and setters (at least `setId`, for `product.setId(...)`).

S3 moves the list and the id counter out of the controller into a service, which is the design your lecturer wants.

---

## 11. Testing your API: three tools

### 11.1 Postman

**Postman** is a desktop app for sending HTTP requests by hand.

| Postman word | Meaning (lecture) |
| :-- | :-- |
| **Workspace** | Your area for organising requests. |
| **Collection** | A folder of related requests, named with a plural noun, e.g. `products`. |
| **Request** | One call: method, URL, parameters, body. |
| **Environment** | Variables you can switch, e.g. `base_url = http://localhost:8080`, used as `{{base_url}}`. |

Sending a POST with a JSON body:

1. **New → HTTP Request.**
2. Method dropdown: **POST**. URL: `{{base_url}}/products` (or the full URL).
3. **Body** tab → **raw** → change the format dropdown from *Text* to **JSON**. (This sets `Content-Type: application/json`. Forgetting it is a common cause of **415 Unsupported Media Type**.)
4. Type the JSON: `{"name": "Keyboard", "price": 59.99}`.
5. **Send**. Read the **status** (top right of the response) and the **body**.

The lecture's collection:

| Action | Method and URL | Body |
| :-- | :-- | :-- |
| Create product | `POST {{base_url}}/products` | `{"name": "Keyboard", "price": 59.99}` |
| Get all | `GET {{base_url}}/products` | none |
| Get by id | `GET {{base_url}}/products/1` | none |
| Delete by id | `DELETE {{base_url}}/products/1` | none |

### 11.2 curl (in a terminal)

```bash
curl -i http://localhost:8081/api/passes
curl -i "http://localhost:8081/api/passes/1/quote?minutes=61"
curl -i -X POST http://localhost:8081/api/passes/hourly \
     -H "Content-Type: application/json" \
     -d '{"id":3,"registration":"CAR-303","rateCents":300}'
```

| Flag | Meaning |
| :-- | :-- |
| `-i` | Show the status line and headers, not only the body. |
| `-X POST` | The method (GET is the default). |
| `-H "..."` | Add a header. |
| `-d '...'` | The body. |
| Quotes around URLs with `?` and `&` | Stops the terminal treating `&` as a command. |

On Windows Command Prompt, use double quotes outside and `\"` inside: `-d "{\"id\":3,\"registration\":\"CAR-303\",\"rateCents\":300}"`. PowerShell users should run `curl.exe` (plain `curl` is a different command there).

### 11.3 IntelliJ HTTP Client (recommended for the exam)

IntelliJ (Ultimate edition, which universities usually provide) can run requests from a file. Create a file ending in **`.http`** (e.g. `requests.http`) in the project root:

```http
### List passes
GET http://localhost:8081/api/passes

### Quote 61 minutes
GET http://localhost:8081/api/passes/1/quote?minutes=61

### Create an hourly pass
POST http://localhost:8081/api/passes/hourly
Content-Type: application/json

{"id":3,"registration":"CAR-303","rateCents":300}
```

| Syntax | Meaning |
| :-- | :-- |
| `###` | Starts a new request (and can carry a title). |
| `GET url` | Method and URL on the first line. |
| Header lines | Straight after, one per line. |
| **One blank line**, then the body | The body must be separated by an empty line. |

A green triangle appears next to each request: click to send. The response opens in a panel with the status and body. You can submit this file inside your ZIP: the paper says "including any test files".

If your IntelliJ is the Community edition and has no HTTP Client, use Postman or curl.

---

## 12. Common errors

| Symptom | Cause | Fix |
| :-- | :-- | :-- |
| `{}` or missing fields in JSON | No getter / misspelled getter | Add `getX()` / `isX()` with exact spelling |
| 400 on POST with valid-looking JSON | Class has no no-arg constructor (and is not a record), or JSON is malformed | Add `public X() {}`; check commas and quotes |
| 400 on GET with a query parameter | Parameter missing or wrong type (`?minutes=abc`) | Send it; or give `defaultValue` |
| 404 on your new endpoint | Not restarted; wrong port; path typo; class outside the main package | Restart; check `application.properties`; check the base path |
| 405 Method Not Allowed | Right path, wrong verb (e.g. GET to a POST endpoint) | Use the correct method |
| 415 Unsupported Media Type | Body sent without `Content-Type: application/json` | Postman: Body → raw → JSON |
| `cannot find symbol: method getId()` on a record | Records use `id()` | `body.id()` |
| A field set in JSON stays `null`/`0` | Key name differs from the property name | Match the names exactly, case included |
| POST returns 200 | No `@ResponseStatus(HttpStatus.CREATED)` | Add it |

---

## 13. Practice

### 13.1 Quick questions (lecture's, plus record questions)

| # | Question | Answer |
| :-- | :-- | :-- |
| 1 | Why does `Product` need a no-arg constructor if it has a full one? | Jackson uses it to build the object before setting fields during deserialisation. |
| 2 | If a field has a getter but no setter, can it change value? | Yes, inside the class. Outside code cannot change it directly. |
| 3 | What does a private field with no getter produce in JSON? | Nothing: it is silently omitted. |
| 4 | `@PathVariable` or `@RequestParam` for "quote for 61 minutes on pass 2"? | Pass id: path variable. Minutes: request param. `/passes/2/quote?minutes=61`. |
| 5 | How do you read `rateCents` from `NewHourlyPass body`? | `body.rateCents()` |
| 6 | Does Jackson need a no-arg constructor for a record? | No. It calls the canonical constructor. |
| 7 | A POST body leaves out `"id"` for `record NewHourlyPass(long id, ...)`. What is `id()`? | `0`. |
| 8 | What makes a POST reply 201? | `@ResponseStatus(HttpStatus.CREATED)` (or a `ResponseEntity`). |
| 9 | Two records with identical values: `a.equals(b)`? | `true`. |

### 13.2 Write the endpoints (no service yet)

Given this record:

```java
public record Book(long id, String title, int pages) {}
```

Write a controller at `/api/books` with an in-memory `List<Book>` and:

1. `GET /api/books` returning all books;
2. `GET /api/books/{id}`;
3. `GET /api/books/long?minPages=300` returning books with at least that many pages (default 300);
4. `POST /api/books` returning 201.

```java
@RestController
@RequestMapping("/api/books")
public class BookController {

    private final List<Book> books = new ArrayList<>();

    @GetMapping
    public List<Book> all() {
        return books;
    }

    @GetMapping("/{id}")
    public Book one(@PathVariable long id) {
        return books.stream()
                .filter(b -> b.id() == id)
                .findFirst()
                .orElse(null);
    }

    @GetMapping("/long")
    public List<Book> longBooks(@RequestParam(defaultValue = "300") int minPages) {
        return books.stream()
                .filter(b -> b.pages() >= minPages)
                .toList();
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Book add(@RequestBody Book book) {
        books.add(book);
        return book;
    }
}
```

Notice: `b.id() == id` uses `==` because both are primitive `long`s (OOP Phase 3: `==` is right for primitives, `.equals()` for `Long`).

**Path clash check:** does `GET /api/books/long` hit `/{id}` with `id = "long"`? No. Spring prefers the more specific, literal path `/long` over the pattern `/{id}`. (If only `/{id}` existed, `"long"` would fail to convert to a number and give 400.)

---

## 14. Glossary

| Term | Meaning |
| :-- | :-- |
| **Accessor (record)** | The generated method named like the component: `id()`. |
| **API** | The set of requests a program accepts. |
| **Body** | The data carried by a request or response. |
| **Canonical constructor** | A record's generated constructor taking every component. |
| **Component (record)** | One of the fields declared in a record's header. |
| **curl** | A command-line tool for sending HTTP requests. |
| **Deserialisation** | JSON → object. |
| **DTO** | Data Transfer Object: a class that only carries request/response data. |
| **`.http` file** | IntelliJ's request file format. |
| **Jackson** | The JSON library in Spring Boot. |
| **JSON** | Text format for structured data. |
| **Path variable** | A placeholder segment in the URL: `{id}`. |
| **Postman** | A desktop app for sending HTTP requests. |
| **Query string** | The part after `?`: `minutes=61&x=y`. |
| **Record** | An immutable data class written in one line. |
| **REST** | A style where URLs name resources and HTTP methods say what to do. |
| **Serialisation** | Object → JSON. |
| **Status code** | The three-digit result of a request (200, 201, 400, 404...). |

---

## 15. Cheat sheet

```java
@RestController
@RequestMapping("/api/things")                                  // base path
public class ThingController {

    @GetMapping                                                  // GET /api/things
    public List<Thing> all() { ... }

    @GetMapping("/{id}")                                         // GET /api/things/5
    public Thing one(@PathVariable long id) { ... }

    @GetMapping("/{id}/score")                                   // GET /api/things/5/score?at=10
    public Map<String, Object> score(@PathVariable long id, @RequestParam int at) {
        return Map.of("id", id, "score", service.score(id, at));
    }

    @GetMapping("/plan")                                         // GET /api/things/plan?cap=60&urgent=true
    public Plan plan(@RequestParam int cap,
                     @RequestParam(defaultValue = "false") boolean urgent) { ... }

    @PostMapping("/special")                                     // POST /api/things/special
    @ResponseStatus(HttpStatus.CREATED)                          // 201
    public Thing add(@RequestBody NewThing body) { ... }         // record: body.id(), body.name()
}

public record NewThing(long id, String name) {}                  // DTO: final fields, id(), name(), equals, toString
```

| Remember | |
| :-- | :-- |
| JSON comes from **getters** (`getX`, `isX`) | No getter → missing field, no error |
| Classes need a **no-arg constructor** for `@RequestBody` | Records do not |
| Record accessors: `id()`, not `getId()` | Records are immutable |
| Path variable = which item; request param = option | Request params are required unless given a default |
| POST → `@ResponseStatus(HttpStatus.CREATED)` | DELETE → `NO_CONTENT` |
| Postman body: raw + **JSON** | Else 415 |

**Next: S3, the service layer, and inheritance and polymorphism inside a Spring API.**
