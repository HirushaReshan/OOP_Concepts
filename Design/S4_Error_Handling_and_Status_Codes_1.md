# S4: Error Handling and HTTP Status Codes in Spring

**Before this:** S3 (controller and service). OOP Phase 6 (exceptions, `try`/`catch`, custom exceptions) is assumed; this guide is its Spring half.
**Goal:** by the end you can make an API answer 400, 404 and 409 on purpose with a consistent JSON body, write a global exception handler, choose the right status for every situation, and validate input in the service so nothing invalid is ever saved.
**Time needed:** about 1.5 hours.

**Built from your files:** *API Error Handling in Spring Boot: Custom Exceptions and Correct HTTP Status Codes* (21 pages), and the `error` and `dto` packages of your three practice starters.

> **Exam link.** Task 3 (20 marks) in every practice paper: *"missing IDs must raise XNotFoundException; map this to HTTP 404 with ApiError(code="NOT_FOUND", ...)"* and *"map IllegalArgumentException to HTTP 400 and ApiError(code="BAD_REQUEST", ...)"*. Section B gives 7 to 8 of its 20 marks to error checks.

---

## 1. The problem: failing badly

The service returns `Optional<Product>`. The controller has two easy options, and both are wrong (lecture):

| Version | Code | Client sees | It claims |
| :-- | :-- | :-- | :-- |
| 1 | `.orElse(null)` | `200 OK`, empty body | "It worked." |
| 2 | `.orElseThrow()` | `500 Internal Server Error` with `"path"` and server details | "Our server crashed." |

**What it should be: `404 Not Found`**, with a small JSON body **you** designed. Lecture: *your error responses are part of your API. Design them, document them and test them like any other endpoint.*

| Unhandled | Handled |
| :-- | :-- |
| 500: wrong claim about whose fault it is | 404: accurate, and the client can act on it |
| Internals (package, class, line) may leak | Nothing internal leaks |
| Shape varies with the failure | Every error has the same predictable shape |

---

## 2. Custom exceptions in Spring

```java
public class ProductNotFoundException extends RuntimeException {

    public ProductNotFoundException(Long id) {
        super("Product not found with id: " + id);
    }

    public ProductNotFoundException(String message) {
        super(message);
    }
}
```

| Piece | Meaning |
| :-- | :-- |
| `extends RuntimeException` | Unchecked: no `throws` clauses anywhere (OOP Phase 6). |
| Two constructors | Overloading: one builds the message from an id, one takes any message. |
| `super(...)` | **Sets `getMessage()`.** Lecture: *forget it and your handler returns an empty message: a very common silent mark loss.* |
| No fields, no getters | *The class name IS the information.* |

The practice starters **already give you** these classes (`PassNotFoundException`, `MissionNotFoundException`, `WorkloadNotFoundException`, `CommitConflictException`). Read their constructors: they take a `long id` or a `String runKey`.

**The service throws:**

```java
public Product getProductById(Long id) {
    return products.stream()
            .filter(p -> p.getId().equals(id))
            .findFirst()
            .orElseThrow(() -> new ProductNotFoundException(id));
}
```

With a `Map`-based repository whose `get` returns `null` for a missing key (your starters):

```java
ParkingPass pass = repo.get(id);
if (pass == null) {
    throw new PassNotFoundException(id);
}
```

**The controller stays one line:** `return service.getProductById(id);`

*The service decides that something is wrong. It does not decide the HTTP response.*

### 2.1 Where the exception goes

1. The controller calls the service and expects a product.
2. The service finds nothing and **throws**.
3. Nothing catches it, so it travels up out of the service and out of the controller method (unchecked, so no compiler complaint).
4. It reaches **Spring's dispatcher** (the part of Spring that runs every request). Without a handler, Spring guesses **500**.

Section 5 adds the handler that turns it into 404.

---

## 3. `ResponseEntity`: the whole response

When a controller method returns a plain object, Spring chooses the status (200). To choose it yourself, return the whole response:

```java
return ResponseEntity
        .status(HttpStatus.OK)
        .body(product);

return ResponseEntity
        .status(HttpStatus.NOT_FOUND)
        .body(error);
```

| Piece | Meaning |
| :-- | :-- |
| `ResponseEntity<T>` | One complete HTTP response: status + headers + a body of type `T`. Import `org.springframework.http.ResponseEntity`. |
| `ResponseEntity<?>` | `?` = "any type of body". Used when one method can return different body types. |
| `.status(HttpStatus.NOT_FOUND)` | Choose the status. `HttpStatus.NOT_FOUND` is a constant for 404: readable and impossible to mistype. |
| `.body(x)` | The body; Jackson turns `x` into JSON. |

Other shortcuts you may see: `ResponseEntity.ok(x)` (200 with body), `ResponseEntity.notFound().build()` (404, no body), `ResponseEntity.noContent().build()` (204), `ResponseEntity.status(HttpStatus.CREATED).body(x)` (201).

`HttpStatus.NOT_FOUND.value()` gives the number `404`.

---

## 4. The obvious fix, and why it fails

```java
@GetMapping("/products/{id}")
public ResponseEntity<?> one(@PathVariable Long id) {
    try {
        Product product = service.getProductById(id);
        return ResponseEntity.status(HttpStatus.OK).body(product);
    } catch (ProductNotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(e.getMessage());
    }
}
```

It works. The lecture's objections:

| Problem | Why |
| :-- | :-- |
| The same `catch` copied into every endpoint | Five endpoints, five copies. Change the error format: edit all five. |
| HTTP concerns buried in the flow | The method's real job is one line; the rest is error plumbing. |

**So: no try/catch in controllers.** One class handles every error.

---

## 5. The global exception handler

### 5.1 The error body

Lecture version, an ordinary class:

```java
public class ErrorResponse {

    private String message;
    private int status;
    private LocalDateTime timestamp;

    public ErrorResponse() {}

    public ErrorResponse(String message, int status) {
        this.message = message;
        this.status = status;
        this.timestamp = LocalDateTime.now();
    }

    // getters (Jackson needs them) and setters
}
```

Practice-paper version, a record (S2 §9):

```java
public record ApiError(String code, String message) {}
```

Both work. **Use whichever the starter gives you, with its exact constructor.** For the papers that is `new ApiError("NOT_FOUND", message)` and `new ApiError("BAD_REQUEST", message)`.

### 5.2 The handler (lecture)

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ProductNotFoundException ex) {
        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse(ex.getMessage(), HttpStatus.NOT_FOUND.value()));
    }
}
```

| Piece | Meaning |
| :-- | :-- |
| (1) `@ControllerAdvice` on the class | "Spring, watch **every** controller for exceptions." |
| (2) `@ExceptionHandler(ProductNotFoundException.class)` on a method | "Call me when this exception escapes a controller." |
| (3) Parameter `ProductNotFoundException ex` | Spring passes in the thrown object, so `ex.getMessage()` gives "Product not found with id: 999". |
| (4) Return `ResponseEntity<ErrorResponse>` | Status 404 + the error body, which Jackson turns into JSON. |
| You never call `handleNotFound()` | Spring does. That is why it can live in its own class. |

### 5.3 `@RestControllerAdvice` (what your starters use)

```java
@RestControllerAdvice
public class ApiExceptionHandler {
    // TODO Task 3: map IllegalArgumentException -> HTTP 400 and PassNotFoundException -> HTTP 404.
}
```

`@RestControllerAdvice` = `@ControllerAdvice` + `@ResponseBody` ("write return values as JSON"). With `ResponseEntity` return types, the two behave the same. Keep whichever annotation the starter already has.

### 5.4 The practice-paper handler (parking Task 3)

```java
package edu.practice.parking.error;

import edu.practice.parking.dto.ApiError;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class ApiExceptionHandler {

    @ExceptionHandler(PassNotFoundException.class)
    public ResponseEntity<ApiError> notFound(PassNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(new ApiError("NOT_FOUND", ex.getMessage()));
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ApiError> badRequest(IllegalArgumentException ex) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                .body(new ApiError("BAD_REQUEST", ex.getMessage()));
    }
}
```

Result for `GET /api/passes/99/quote?minutes=61`:

```
HTTP/1.1 404
{"code":"NOT_FOUND","message":"Parking pass not found: 99"}
```

The starters import `org.springframework.http.*` and `org.springframework.web.bind.annotation.*` already (the `*` imports everything in the package), so the explicit imports above are optional there.

### 5.5 What Spring does, step by step (lecture)

1. The service throws; your code stops. No `catch` in service or controller.
2. Spring's dispatcher catches it at the boundary.
3. Spring searches the `@ControllerAdvice` / `@RestControllerAdvice` classes for an `@ExceptionHandler` whose type matches. **The most specific match wins**, like a chain of `catch` blocks.
4. Your handler returns a `ResponseEntity`; Spring sends it.

"Most specific wins" matters:

| Thrown | Handlers present | Which runs |
| :-- | :-- | :-- |
| `NumberFormatException` | `IllegalArgumentException` | The `IllegalArgumentException` one: `NumberFormatException` is its child. |
| `PassNotFoundException` | `PassNotFoundException` and `RuntimeException` | `PassNotFoundException`'s. |
| `ArithmeticException` (overflow) | only `IllegalArgumentException`, `PassNotFoundException` | **None** → Spring's 500. So convert overflow to `IllegalArgumentException` yourself (S6). |

---

## 6. HTTP status codes: the seven you need

| Code | Name | Use for | Whose fault |
| :-- | :-- | :-- | :-- |
| **200** | OK | Successful GET (or a PUT returning the updated object) | Nobody's |
| **201** | Created | A POST that created something. Return the new object, so the client learns its id. | Nobody's |
| **204** | No Content | A successful DELETE. No body. | Nobody's |
| **400** | Bad Request | The client sent something invalid. Fixable by them. | Client |
| **404** | Not Found | The resource does not exist. | Client |
| **409** | Conflict | Valid request, but it clashes with current state: a duplicate id, a reused key with different data | Client |
| **500** | Internal Server Error | **You** broke. If a client can trigger it on purpose, it should have been a 4xx. | You |

### 6.1 Three ways to set a status (lecture)

| Way | Code | When |
| :-- | :-- | :-- |
| 1. `ResponseEntity` in the handler | `ResponseEntity.status(HttpStatus.NOT_FOUND).body(new ErrorResponse(msg, 404))` | Inside `@ExceptionHandler`: the only form that also sets a custom body. |
| 2. `@ResponseStatus` on a controller method | `@PostMapping @ResponseStatus(HttpStatus.CREATED)` | Success codes: 201 for POST, 204 for DELETE. |
| 3. `@ResponseStatus` on the exception class | `@ResponseStatus(HttpStatus.NOT_FOUND) public class ProductNotFoundException ...` | Works, but hides the status and cannot set a body. Prototype only. |

Never hand-write numbers where a constant exists: `HttpStatus.CREATED` beats `201`.

### 6.2 Who decides the status (lecture's rule)

| Class | Decides |
| :-- | :-- |
| **Service** | **If** something is wrong. Throws. Never mentions HTTP. |
| **Controller** | The **success** status: nothing for 200, `@ResponseStatus` for 201/204. |
| **Exception handler** | Every **error** status: 400, 404, 409. |

---

## 7. Validation in the service

**Validation** = checking input before using it. In the practice papers, validation failures throw `IllegalArgumentException`, which the handler turns into 400.

The parking paper's `addHourly` rules: *positive unique id, non-blank registration, positive hourly rate; trim registration before storing.*

```java
public ParkingPass addHourly(NewHourlyPass body) {
    if (body == null) {
        throw new IllegalArgumentException("Request body is required");
    }
    if (body.id() <= 0) {
        throw new IllegalArgumentException("id must be a positive number");
    }
    if (body.registration() == null || body.registration().isBlank()) {
        throw new IllegalArgumentException("registration is required");
    }
    if (body.rateCents() <= 0) {
        throw new IllegalArgumentException("rateCents must be greater than zero");
    }
    if (repo.contains(body.id())) {
        throw new IllegalArgumentException("A pass with id " + body.id() + " already exists");
    }
    HourlyPass pass = new HourlyPass(body.id(), body.registration().trim(), body.rateCents());
    repo.save(pass);                 // only reached if every check passed
    return pass;
}
```

| Habit | Why |
| :-- | :-- |
| One `if` per rule, each with its own message | Partial marks per rule; the client sees exactly what was wrong. |
| `== null` **before** `.isBlank()` | Short-circuit `\|\|` stops a `NullPointerException` (OOP Phase 2). |
| Every check **before** `save` | Section B: *"returns 400 without persisting a pass"*. |
| `trim()` before storing | `"  CAR-303 "` becomes `"CAR-303"`. `trim()` removes spaces at both ends. |
| Duplicate id | The papers group it with validation (400). In a lecture-style API a duplicate is often 409 Conflict; follow the paper's wording. |

### 7.1 Spring's own 400s

Some bad requests never reach your code:

| Request | Spring throws | Default result |
| :-- | :-- | :-- |
| `?minutes=` missing | `MissingServletRequestParameterException` | 400, Spring's body |
| `?minutes=abc` | `MethodArgumentTypeMismatchException` | 400, Spring's body |
| Broken JSON, or a number field sent as `"abc"` | `HttpMessageNotReadableException` | 400, Spring's body |

Already 400, so Section B's "returns 400" passes. To give them your `ApiError` shape too, add one more handler:

```java
import org.springframework.http.converter.HttpMessageNotReadableException;
import org.springframework.web.bind.MissingServletRequestParameterException;
import org.springframework.web.method.annotation.MethodArgumentTypeMismatchException;

@ExceptionHandler({HttpMessageNotReadableException.class,
        MissingServletRequestParameterException.class,
        MethodArgumentTypeMismatchException.class})
public ResponseEntity<ApiError> unreadable(Exception ex) {
    return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(new ApiError("BAD_REQUEST", "Malformed or missing request data"));
}
```

`@ExceptionHandler({A.class, B.class})` handles several types; the parameter is then their common parent (`Exception`). Optional, but it makes every 400 look the same.

---

## 8. Practical: add 404 and 400 to an API

1. Write (or find) the custom not-found exception; check its constructor calls `super(...)`.
2. In the service, throw it where the item is missing (`orElseThrow` or `if (x == null)`).
3. In the service, add `IllegalArgumentException` checks **before** any save.
4. In the handler class (`@ControllerAdvice` or `@RestControllerAdvice`), add one `@ExceptionHandler` per exception type, each returning `ResponseEntity.status(...).body(new ApiError(...))`.
5. Restart.
6. Test, reading the **status** as well as the body:

```bash
curl -i "http://localhost:8081/api/passes/99/quote?minutes=61"     # expect 404 + NOT_FOUND
curl -i "http://localhost:8081/api/passes/1/quote?minutes=0"       # expect 400 + BAD_REQUEST
curl -i -X POST http://localhost:8081/api/passes/hourly -H "Content-Type: application/json" \
     -d '{"id":3,"registration":"   ","rateCents":300}'            # expect 400
curl -i http://localhost:8081/api/passes                           # pass 3 must NOT be there
```

Still 500? Check: the handler annotation is present; the `@ExceptionHandler` type matches what is thrown; the handler is in a package below the main class; you restarted.

---

## 9. Common mistakes

| Mistake | Effect | Fix |
| :-- | :-- | :-- |
| No `@ControllerAdvice`/`@RestControllerAdvice` | Handler never runs; 500 | Add the annotation |
| Exception without `super(message)` | `"message": null` | Call `super(...)` |
| try/catch in the controller | Duplication; breaks the layering | Let it propagate |
| Validation after `save` | Invalid data stored | Validate first |
| `orElse(null)` in the controller | 200 with empty body | Throw in the service |
| Wrong `ApiError` code text | Loses the "correct error code" mark | Exactly `"NOT_FOUND"`, `"BAD_REQUEST"`, `"CONFLICT"` as the paper says |
| Overflow left as `ArithmeticException` | 500 | Catch and throw `IllegalArgumentException` |
| 400 for a missing resource / 404 for invalid data | Wrong codes | 404 = does not exist; 400 = invalid input |
| POST returns 200 | Wrong code | `@ResponseStatus(HttpStatus.CREATED)` |

---

## 10. Practice questions

| # | Question | Answer |
| :-- | :-- | :-- |
| 1 | Why is `.orElse(null)` wrong for a missing product? | The client gets 200 and thinks it worked. |
| 2 | Why is plain `.orElseThrow()` wrong? | 500 claims the server broke, and may leak internals. |
| 3 | What does `@ExceptionHandler(X.class)` do? | Marks a method Spring calls when `X` escapes a controller. |
| 4 | Difference between `@ControllerAdvice` and `@RestControllerAdvice`? | The second adds `@ResponseBody`; with `ResponseEntity` they behave the same. |
| 5 | Which layer throws, which sets success codes, which sets error codes? | Service; controller; exception handler. |
| 6 | Status for a successful POST that creates? DELETE? | 201; 204. |
| 7 | Status for a reused key with different data (EXTREME paper)? | 409 Conflict. |
| 8 | A `NumberFormatException` is thrown; you only handle `IllegalArgumentException`. Result? | Your 400 handler runs: it is a subclass. |
| 9 | Your POST validation passes four rules, then the fifth fails. Was anything saved? | Not if `save` comes after all five checks. |
| 10 | What sets `ex.getMessage()`? | The `super(...)` call in the exception's constructor. |

---

## 11. Cheat sheet

```java
// exception (often provided)
public class ThingNotFoundException extends RuntimeException {
    public ThingNotFoundException(long id) { super("Thing not found: " + id); }
}

// service: decide IF wrong, validate BEFORE saving
Thing t = repo.get(id);
if (t == null) throw new ThingNotFoundException(id);
if (body.name() == null || body.name().isBlank()) throw new IllegalArgumentException("name is required");

// handler: every error status in one class
@RestControllerAdvice
public class ApiExceptionHandler {
    @ExceptionHandler(ThingNotFoundException.class)
    public ResponseEntity<ApiError> notFound(ThingNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(new ApiError("NOT_FOUND", ex.getMessage()));
    }
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ApiError> badRequest(IllegalArgumentException ex) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(new ApiError("BAD_REQUEST", ex.getMessage()));
    }
    @ExceptionHandler(ConflictException.class)
    public ResponseEntity<ApiError> conflict(ConflictException ex) {
        return ResponseEntity.status(HttpStatus.CONFLICT).body(new ApiError("CONFLICT", ex.getMessage()));
    }
}

// controller: success status only
@PostMapping("/x") @ResponseStatus(HttpStatus.CREATED)
public Thing add(@RequestBody NewThing body) { return service.add(body); }
```

| Remember | |
| :-- | :-- |
| 200 GET · 201 POST · 204 DELETE | 400 invalid · 404 missing · 409 conflict · 500 your bug |
| Service throws; handler maps; controller has no try/catch | Most specific handler wins |
| `super(message)` in every exception | Exact `ApiError` code strings |
| Validate first, save last | Convert overflow to `IllegalArgumentException` |

**Next: S5, the repository layer, IoC and DI in Spring, and testing.**
