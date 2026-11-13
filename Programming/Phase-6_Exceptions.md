# Phase 6: Exceptions

**Module:** 5COSC019W Object Oriented Programming
**Before this:** Phase 5 (overriding, abstract classes, interfaces). Phase 3's `Optional` is used heavily.
**Goal:** by the end you can explain the exception family tree, use `try` / `catch` / `finally` correctly, write your own exception classes, throw them from a service, and know how they become proper `404` and `400` responses in your exam project.
**Time needed:** about 2 hours.

**Built from your files:** Lecture 6 (Exceptions: Failing on Purpose), Sections 01–06, plus its mock-exam briefing.

> **Why this phase matters for the exam.** Task 3 is *"Services, validation and exceptions"* (20 marks). In the mock it was *"a custom exception thrown in the service, turned into 404 by a global handler"*, and the harder Task 5 was *"a new exception for invalid data, and a second handler, returning 400"*. Three of the six Section B checks are about errors: unknown id gives 404 (not 500), the 404 body names the id, and invalid data gives 400 with nothing saved.

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
| **Phase 6** | **Exceptions** | **You are here** |
| Phase 7 | Dependencies and Layered Design | Next |
| Phase 8 | Exam Consolidation | |

---

## 1. The problem: failing badly

Phase 3's service returns `Optional<Product>` from `getProductById`. Lecture 6 shows the two ways the API handled a missing product, and why both are wrong.

| Version | Code | What the client got | What it claims |
| :-- | :-- | :-- | :-- |
| 1 | `.orElse(null)` | `200 OK`, empty body | "It worked!" (It did not.) |
| 2 | `.orElseThrow()` | `500 Internal Server Error`, plus a stack trace naming your package, class and line number | "Our server has crashed." (It had not.) |

Lecture 6's key takeaway: *nothing here is broken. A client asked for something that does not exist. First we told them it worked, then we told them our server had crashed.*

The honest answer is **"404 Not Found"**: that thing does not exist. To give it, the program must **fail on purpose**: notice the problem, describe it clearly, and let one place decide how to report it. Java's tool for that is the **exception**.

---

## 2. What an exception is

An **exception** is an **object** that describes a problem, created and **thrown** at the moment the problem is found. Throwing it **stops the normal flow** of the program immediately, and the exception travels **back up** through the methods that were called, until some code **catches** it and deals with it. If nothing catches it, the program (or, in Spring, the request) ends with an error.

- **Analogy: a fire alarm.** Someone notices smoke (the problem) and pulls the alarm (throws). Normal work stops at once. The alarm travels up the building to whoever is responsible for fires (the catcher). If nobody responds, the whole building is evacuated (the program crashes).

You have already met several:

| Exception | Phase | Thrown when |
| :-- | :-- | :-- |
| `ArithmeticException` | Pre-0 | An `int` is divided by zero |
| `ArrayIndexOutOfBoundsException` | Pre-0 | An array position does not exist |
| `NullPointerException` | 1 | The dot is used on `null` |
| `IllegalArgumentException` | 2 | A method is handed an unacceptable value |
| `IllegalStateException` | 2 | An action is not allowed in the object's current state |
| `IndexOutOfBoundsException` | 3 | A list position does not exist |
| `NoSuchElementException` | 3 | `get()` on an empty `Optional` |
| `ClassCastException` | 5 | A cast is wrong |

### 2.1 Reading an uncaught exception

```
Exception in thread "main" java.lang.IllegalArgumentException: Price cannot be negative
	at Product.setPrice(Product.java:21)
	at Product.<init>(Product.java:12)
	at Main.main(Main.java:5)
```

| Part | Meaning |
| :-- | :-- |
| `java.lang.IllegalArgumentException` | The **type**: which class of problem. |
| `Price cannot be negative` | The **message**: the human explanation. |
| `at Product.setPrice(Product.java:21)` | Where it was thrown: method, file, line. |
| The lines below | The chain of calls that led there, most recent first. `<init>` means "a constructor". |

This list of calls is the **stack trace** (Pre-Phase 0, Section 13). Read from the top, and find the first line in **your** code.

---

## 3. Exceptions are objects in a class hierarchy

Lecture 6: *every failure in Java is an object, in a class hierarchy you already understand* (Phase 4).

```
                         Throwable
                        /         \
                    Error          Exception
             (never catch)         /         \
                          IOException      RuntimeException
                          (checked)         (unchecked)
                                          /       |        \
                      IllegalArgumentException   NullPointer...   ProductNotFoundException
                                                                       (yours)
```

| Class | Meaning |
| :-- | :-- |
| `Throwable` | The top. Anything that can be thrown. |
| `Error` | Serious JVM problems, such as `OutOfMemoryError` or `StackOverflowError`. **Never catch these.** Nothing your code does can fix them. |
| `Exception` | Problems a program might reasonably deal with. |
| `RuntimeException` | A special branch of `Exception`: mostly programming mistakes and rule violations. |
| Your own exceptions | Classes you write, normally extending `RuntimeException`. |

Because these are ordinary classes, everything from Phases 4 and 5 applies: a `ProductNotFoundException` **IS-A** `RuntimeException`, which IS-A `Exception`. A `catch` for a parent type also catches every child type.

### 3.1 Checked versus unchecked

| | **Checked** | **Unchecked** |
| :-- | :-- | :-- |
| Which | `Exception` and its subclasses **except** `RuntimeException` (e.g. `IOException`, `SQLException`) | `RuntimeException` and all its subclasses (e.g. `NullPointerException`, `IllegalArgumentException`) |
| Typical cause | Something **outside** your program failed: a missing file, a lost network | A programming mistake or broken rule inside your program |
| Compiler | **Insists.** Every method that might throw one must either catch it or declare `throws` | **Silent.** No `throws` needed anywhere |

A checked exception and the `throws` declaration look like this:

```java
public String readFirstLine(String path) throws IOException {
    ...
}
```

`throws IOException` in the header warns callers: "this can fail with an `IOException`; you must deal with it." Every caller must then catch it or add `throws` too, all the way up. Forget, and the code does not compile.

**Lecture 6's advice: prefer unchecked exceptions in Spring.** A checked exception forces `throws` onto every layer between service and controller, and Spring's handlers catch both kinds equally well. **Your own exceptions should extend `RuntimeException`.**

Lecture 6 quick checks:

| Question | Answer |
| :-- | :-- |
| Is `NullPointerException` checked or unchecked? | Unchecked: it extends `RuntimeException`. That is exactly why it reaches production. |
| Your service throws `ProductNotFoundException extends RuntimeException`. Does the controller need a `throws` clause? | No. Unchecked exceptions travel up freely. |
| Should you ever catch `Error`, say `OutOfMemoryError`? | No. Your catch block has nothing useful to do. |

---

## 4. `try`, `catch`, `finally`

### 4.1 Catching an exception

```java
String raw = "abc";
try {
    int credits = Integer.parseInt(raw);
    System.out.println("Credits: " + credits);
} catch (NumberFormatException e) {
    System.out.println("Not a number: " + raw);
}
System.out.println("Program continues");
```

**Output:**

```
Not a number: abc
Program continues
```

| Piece | What it means |
| :-- | :-- |
| `try {` | "Attempt the code in this block." |
| `Integer.parseInt(raw)` | A built-in method that turns text like `"20"` into the `int` 20. With `"abc"` it throws `NumberFormatException`. |
| `System.out.println("Credits: ...")` | **Skipped.** The moment an exception is thrown, the rest of the `try` block is abandoned. |
| `catch (NumberFormatException e) {` | "If an exception of this type is thrown inside the `try`, run this block instead." `e` is a variable holding the exception object. |
| After the `catch` | The program carries on normally. The exception has been **handled**. |

- **Analogy: a safety net under a trapeze act.** The performer attempts the trick (`try`). If they fall (exception), the net catches them (`catch`) and the show continues. If they do not fall, the net is never used.

Useful methods on the exception object `e`:

| Method | Returns |
| :-- | :-- |
| `e.getMessage()` | The message text. |
| `e.printStackTrace()` | Prints the full stack trace (useful while debugging). |

### 4.2 Several catch blocks: most specific first

```java
try {
    ...
} catch (NumberFormatException e) {
    ...
} catch (IllegalArgumentException e) {
    ...
}
```

Catch blocks are tested **top to bottom**, and the first matching one runs. `NumberFormatException` is a child of `IllegalArgumentException`, so it must come **first**. The other way round, the parent would catch everything and the child's block could never run; the compiler rejects that ("exception has already been caught").

**Multi-catch** handles two unrelated types in one block, joined by `|`:

```java
} catch (IllegalStateException | IllegalArgumentException e) {
    System.out.println("Rejected: " + e.getMessage());
}
```

### 4.3 `finally`: always runs

```java
try {
    ...
} catch (SomeException e) {
    ...
} finally {
    System.out.println("Always printed");
}
```

`finally` runs **on every exit path**: whether the `try` finished normally, an exception was caught, an exception escaped, or a `return` happened. It is where you put clean-up work: closing files, releasing resources, writing an audit record.

Lecture 6's "predict the output":

```java
public static String check() {
    try {
        return "A";
    } finally {
        System.out.println("B");
    }
}

System.out.println(check());
```

**Output:**

```
B
A
```

`"A"` is worked out and held, `finally` runs and prints `B`, and only then does the method hand back `"A"` to be printed.

**Never put `return` inside `finally`.** It throws away the held value and silently swallows any exception in flight.

### 4.4 Re-throwing and translating

A catch block may throw again: either the same exception (`throw e;`), or a more meaningful one. Turning a low-level exception into a domain one is called **translating**:

```java
try {
    int credits = Integer.parseInt(raw);
    ...
} catch (NumberFormatException e) {
    throw new InvalidCreditsException(raw);
}
```

"The text was not a number" becomes "the credits value is invalid", which the rest of the program understands.

### 4.5 Three ways to get it wrong (Lecture 6)

| Mistake | Code | Why it is bad |
| :-- | :-- | :-- |
| **Swallowing** | `catch (Exception e) { }` | The failure vanishes. The bug resurfaces later somewhere unrelated, with no trace back. The hardest kind of defect to find. |
| **Casting too wide** | `catch (Exception e)` | You meant to catch a parse failure, and also caught a `NullPointerException` that was telling you about a real bug, and hid it. |
| **Catching where you cannot act** | catch, print, return something wrong | The method cannot fix the problem, so it should not pretend to. Let it travel to a layer that can decide. |

**Lecture 6's practical rule: catch only where you can do something useful about it. Everywhere else, let it travel.**

---

## 5. Throwing exceptions yourself

```java
throw new IllegalArgumentException("Price cannot be negative");
```

| Piece | What it means |
| :-- | :-- |
| `throw` | "Stop now and send this exception up." |
| `new IllegalArgumentException(...)` | Build the exception object. It is an ordinary object built with `new`. |
| `"Price cannot be negative"` | The message, available later through `getMessage()`. |

You used this in Phase 2 setters. The method ends immediately: no code after `throw` in that block runs, and no value is returned.

Do not confuse the two keywords:

| `throw` | `throws` |
| :-- | :-- |
| A statement inside a method body | Part of a method header |
| Actually throws one exception now | Warns that the method **might** throw these types |
| `throw new X("...");` | `void m() throws IOException` |

### 5.1 How an exception travels

```java
public class Propagation {

    static void level3() {
        throw new IllegalStateException("Problem found in level 3");
    }

    static void level2() {
        level3();
        System.out.println("level2 finished");      // never runs
    }

    static void level1() {
        try {
            level2();
        } catch (IllegalStateException e) {
            System.out.println("Caught in level1: " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        level1();
        System.out.println("main continues");
    }
}
```

**Output:**

```
Caught in level1: Problem found in level 3
main continues
```

| Step | What happens |
| :-- | :-- |
| 1 | `main` → `level1` → `level2` → `level3`. |
| 2 | `level3` throws. It ends at once. |
| 3 | `level2` has no `catch`, so it ends too, mid-way. `"level2 finished"` never prints. |
| 4 | `level1` has a matching `catch`. It handles the exception. |
| 5 | Normal flow resumes after the `try`/`catch` in `level1`, and `main` continues. |

Travelling up through methods like this is called **propagation** (or "unwinding the call stack"). This is exactly how a service's exception reaches Spring's handler without the controller doing anything.

---

## 6. Custom exception classes

### 6.1 Why not just throw `RuntimeException`?

| Generic | Custom |
| :-- | :-- |
| `throw new RuntimeException("not found")` | `throw new ProductNotFoundException(id)` |
| The message string is the only clue. | **The class name states the problem** before you read any message. |
| A handler would have to match on text, and text changes. | A handler can target this exact type and nothing else. |
| The stack trace says `RuntimeException`, which describes nothing. | Logs name it, so failures are searchable. |

Lecture 6's OOP point: *a custom exception is just a class in an inheritance hierarchy, the same mechanism from Lecture 4, used to make failure part of your design.*

### 6.2 Five lines doing real work

```java
public class ProductNotFoundException extends RuntimeException {

    public ProductNotFoundException(Long id) {
        super("Product not found with id: " + id);
    }
}
```

| Line | What it means |
| :-- | :-- |
| `public class ProductNotFoundException` | Exception class names end in `Exception` by convention. |
| `extends RuntimeException` | It IS-A `RuntimeException`, so it is **unchecked**: no `throws` needed anywhere. |
| `public ProductNotFoundException(Long id) {` | The constructor takes the **id**, not a ready-made message. |
| `super("Product not found with id: " + id);` | Passes the message up to `RuntimeException`'s constructor. **This is what sets `getMessage()`.** |

No fields, no getters. **The class name is the information.**

> **Lecture 6's warning:** *"`super(...)` is what sets `getMessage()`. Forget it and your handler returns an empty message, a very common silent mark loss."* Without it, `getMessage()` returns `null`, and Section B's check that *"the 404 body names the id"* fails.

Lecture 6, Q2: *"The constructor takes a `Long id`, not a `String` message. Why is that better?"* **The class owns its wording, so every throw site produces the same correctly formatted message, and the id is never forgotten.**

### 6.3 A validation exception

For the "400 Bad Request" kind of failure (invalid data from the client):

```java
public class InvalidProductException extends RuntimeException {

    public InvalidProductException(String message) {
        super(message);
    }
}
```

Here the constructor takes a message, because there are several different ways data can be invalid ("Name is required", "Price must be positive").

### 6.4 Lecture 6's design activity

| Failure | Exception | Status |
| :-- | :-- | :-- |
| `GET /courses/999`: no course with that id | `CourseNotFoundException` | 404 Not Found |
| `POST /courses` with credits = -20 | `InvalidCreditsException` | 400 Bad Request |
| `POST /courses` with an id that already exists | `DuplicateCourseException` | 409 Conflict |

---

## 7. Where the throw belongs: in the service

### 7.1 `orElseThrow` with your own exception

Phase 3's `getProductById` returned `Optional<Product>`. Now the service **decides** that a missing product is an error:

```java
public Product getProductById(Long id) {
    return products.stream()
            .filter(p -> p.getId().equals(id))
            .findFirst()
            .orElseThrow(() -> new ProductNotFoundException(id));
}
```

| Piece | Meaning |
| :-- | :-- |
| Return type `Product` | No longer `Optional`. The caller either gets a real product or an exception. |
| `.orElseThrow(...)` | If the `Optional` is empty, throw. |
| `() -> new ProductNotFoundException(id)` | A lambda with **no input** (the empty `()`) that builds the exception. `orElseThrow` only calls it if it needs to. |

### 7.2 Validation in the service

```java
public Product addProduct(Product product) {
    if (product.getName() == null || product.getName().isBlank()) {
        throw new InvalidProductException("Name is required");
    }
    if (product.getPrice() <= 0) {
        throw new InvalidProductException("Price must be greater than zero");
    }
    product.setId(nextId++);
    products.add(product);
    return product;
}
```

The checks come **before** anything is saved. If one fails, the method stops, so **nothing is saved**. That is Section B's last check: *"POST invalid: 400 Bad Request with a JSON message, and nothing is saved."*

### 7.3 Who does what (Lecture 6)

| Layer | Its job with errors |
| :-- | :-- |
| **Service** | **Decides if** something is wrong. Owns the business rules. Throws. Never mentions HTTP. |
| **Controller** | Sets the **success** status. **Never catches, never throws.** One line per method. |
| **Global handler** | Sets every **error** status: 404, 400, 409... in one class. |

Lecture 6, Q3: *"Should the service catch the `ProductNotFoundException` it just threw?"* **No. It threw it precisely because it could not fix the situation.**

### 7.4 The whole flow in plain Java

To see the pattern without Spring, here `main` plays the role of the handler: one place that catches and reports.

```java
public class ShopDemo {

    public static void main(String[] args) {
        ProductService service = new ProductService();
        service.addProduct(new Product(null, "Laptop", 899.99));

        try {
            System.out.println(service.getProductById(1L).getName());
            System.out.println(service.getProductById(999L).getName());
        } catch (ProductNotFoundException e) {
            System.out.println("404: " + e.getMessage());
        }

        try {
            service.addProduct(new Product(null, "Mouse", -5));
        } catch (InvalidProductException e) {
            System.out.println("400: " + e.getMessage());
        }

        System.out.println("Products saved: " + service.getAllProducts().size());
    }
}
```

**Output:**

```
Laptop
404: Product not found with id: 999
400: Price must be greater than zero
Products saved: 1
```

The invalid mouse was never saved: still 1 product.

---

## 8. How this becomes 404 and 400 in your exam project

This is Spring code, which your Spring chat will cover in depth. But Task 3 needs it, so here is exactly enough to write it.

### 8.1 Three Spring pieces

| Piece | Plain meaning |
| :-- | :-- |
| `ResponseEntity<T>` | One complete HTTP response: status code + headers + a body of type `T`. Lets you choose the status. |
| `@ControllerAdvice` | Written above a class: "Spring, this class watches **every** controller for exceptions." |
| `@ExceptionHandler(X.class)` | Written above a method: "call me when an `X` escapes a controller." |

`X.class` means "the class `X` itself" (as a value), rather than an object of it.

### 8.2 The error body

```java
import java.time.LocalDateTime;

public class ErrorResponse {

    private String message;
    private int status;
    private LocalDateTime timestamp;

    public ErrorResponse(String message, int status) {
        this.message = message;
        this.status = status;
        this.timestamp = LocalDateTime.now();
    }

    public String getMessage() { return message; }
    public int getStatus() { return status; }
    public LocalDateTime getTimestamp() { return timestamp; }
}
```

An ordinary encapsulated class (Phase 2). **The getters are essential**: Jackson reads them to build the JSON. `LocalDateTime.now()` is the current date and time. In the mock, this class was **provided**: read it to find its constructor's parameters.

### 8.3 The handler (Lecture 6's recipe)

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ProductNotFoundException ex) {
        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse(ex.getMessage(), 404));
    }

    @ExceptionHandler(InvalidProductException.class)
    public ResponseEntity<ErrorResponse> handleInvalid(InvalidProductException ex) {
        return ResponseEntity
                .status(HttpStatus.BAD_REQUEST)
                .body(new ErrorResponse(ex.getMessage(), 400));
    }
}
```

| Line | Meaning |
| :-- | :-- |
| `@ControllerAdvice` | Register this class with Spring. **Forget it and the class never runs**; you get a 500. |
| `@ExceptionHandler(ProductNotFoundException.class)` | This method handles that exception type. One method per type. |
| `handleNotFound(ProductNotFoundException ex)` | Spring passes in the thrown exception. You never call this method yourself. |
| `ResponseEntity.status(HttpStatus.NOT_FOUND)` | Status 404. `HttpStatus.NOT_FOUND` is a named constant: cannot be mistyped. |
| `.body(new ErrorResponse(ex.getMessage(), 404))` | The JSON body. `ex.getMessage()` is the text set by `super(...)` in your exception. |

Lecture 6's six-step recipe: (1) create `GlobalExceptionHandler` in the controller's package; (2) `@ControllerAdvice`; (3) `@ExceptionHandler(X.class)` above a method; (4) parameter `X ex`; (5) return `ResponseEntity.status(...).body(...)`; (6) restart and test with id 999: 404 and your JSON. Still 500? Go back to step 2.

The result for `GET /products/999`:

```json
{
  "message": "Product not found with id: 999",
  "status": 404,
  "timestamp": "2026-02-19T14:22:07.881"
}
```

### 8.4 The status codes you need (Lecture 6)

| Code | Name | Use for |
| :-- | :-- | :-- |
| **200** | OK | Successful GET (Spring's default) |
| **201** | Created | A POST that created something. Return the new object, so the client learns its id. |
| **204** | No Content | A successful DELETE. No body. |
| **400** | Bad Request | The client sent invalid data. Their fault, fixable by them. |
| **404** | Not Found | The thing does not exist. |
| **409** | Conflict | Valid request, but it clashes with current state, e.g. a duplicate id. |
| **500** | Internal Server Error | **You** broke. If a client can cause it on purpose, it should have been a 4xx. |

Lecture 6's test: **whose fault is it?** 2xx: nobody's. 4xx: the client's. 5xx: yours.

On the success side, the controller sets 201 and 204 with one annotation, `@ResponseStatus(HttpStatus.CREATED)` or `@ResponseStatus(HttpStatus.NO_CONTENT)`, above the method. A method returning a plain object otherwise always gives 200, and *"returning 200 from POST"* is on Lecture 6's list of where people lose time.

---

## 9. Common mistakes

| Mistake | What happens | Fix |
| :-- | :-- | :-- |
| Custom exception without `super(message)` | `getMessage()` is `null`; the 404 body has no message. | `super("... " + id);` |
| `extends Exception` instead of `RuntimeException` | Checked: `throws` needed on every method up the chain. | `extends RuntimeException` |
| try/catch in the controller | Duplicate code; handler never runs; breaks the layer rule. | Let it propagate to the handler. |
| Validation **after** saving | Invalid item is stored before the exception. | Validate first, then save. |
| `catch (Exception e) { }` | Failure swallowed silently. | Catch specific types, only where you can act. |
| Parent `catch` before child `catch` | Compile error: "has already been caught". | Most specific first. |
| `return` inside `finally` | Swallows exceptions and overrides the real return. | Never do it. |
| Missing `@ControllerAdvice` | Handler never registered; still 500. | Add it. |
| Returning 400 for a missing resource | Wrong code. | 404 for "does not exist", 400 for "invalid data". |
| `.orElse(null)` | `200` with an empty body for a missing item. | `.orElseThrow(() -> new XNotFoundException(id))` |
| Catching `Error` | Pointless; can make things worse. | Never. |

---

## 10. Practice

### 10.1 Predict the output

```java
// A
try {
    System.out.println("1");
    int x = 10 / 0;
    System.out.println("2");
} catch (ArithmeticException e) {
    System.out.println("3");
} finally {
    System.out.println("4");
}
System.out.println("5");
```

```java
// B
try {
    System.out.println("1");
} catch (Exception e) {
    System.out.println("2");
} finally {
    System.out.println("3");
}
```

```java
// C
try {
    String s = null;
    s.length();
} catch (IllegalArgumentException e) {
    System.out.println("caught");
} finally {
    System.out.println("finally");
}
System.out.println("after");
```

**Answers**

| | Output | Why |
| :-- | :-- | :-- |
| A | `1`, `3`, `4`, `5` | `10 / 0` throws; `2` is skipped; the catch matches; `finally` always runs; then normal flow continues. |
| B | `1`, `3` | No exception, so the catch never runs; `finally` still does. |
| C | `finally`, then the program crashes with `NullPointerException` | The catch is for the wrong type, so it does not match. `finally` still runs, then the exception escapes. `after` never prints. |

### 10.2 Quick questions

| # | Question | Answer |
| :-- | :-- | :-- |
| 1 | Difference between `Error` and `Exception`? | `Error` is a serious JVM problem you never catch; `Exception` is something a program may handle. |
| 2 | Checked vs unchecked? | Checked: the compiler forces catch-or-declare. Unchecked (`RuntimeException` family): it does not. |
| 3 | Why should custom exceptions extend `RuntimeException`? | No `throws` clauses needed through every layer; Spring handles them the same. |
| 4 | What sets `getMessage()` in a custom exception? | The call `super(message)` in its constructor. |
| 5 | Which layer throws `ProductNotFoundException`? | The service. |
| 6 | Which layer turns it into a 404? | The global exception handler. |
| 7 | DELETE succeeds. 200 or 204? | 204 No Content. |
| 8 | POST creates something. Which code? | 201 Created, with the saved object. |
| 9 | `throw` vs `throws`? | `throw` throws one now; `throws` in a header warns it might. |

### 10.3 Build it: `CourseService` with two exceptions

Using this `Course`:

```java
public class Course {

    private Long id;
    private String title;
    private int credits;

    public Course(Long id, String title, int credits) {
        this.id = id;
        this.title = title;
        this.credits = credits;
    }

    public Long getId() { return id; }
    public String getTitle() { return title; }
    public int getCredits() { return credits; }
    public void setId(Long id) { this.id = id; }
}
```

Write:

1. `CourseNotFoundException` (takes the id; message `"Course not found with id: <id>"`);
2. `InvalidCreditsException` (takes the bad number; message `"Credits must be a positive number, got: <n>"`);
3. `CourseService` with `addCourse` (rejects credits ≤ 0 **before** saving), `findById` (throws if missing) and `getAll`;
4. a `main` that adds one valid course, tries an invalid one, looks up a missing id, and prints how many courses were saved.

Try for 20 minutes.

```java
public class CourseNotFoundException extends RuntimeException {

    public CourseNotFoundException(Long id) {
        super("Course not found with id: " + id);
    }
}
```

```java
public class InvalidCreditsException extends RuntimeException {

    public InvalidCreditsException(int credits) {
        super("Credits must be a positive number, got: " + credits);
    }
}
```

```java
import java.util.ArrayList;
import java.util.List;

public class CourseService {

    private List<Course> courses = new ArrayList<>();
    private Long nextId = 1L;

    public Course addCourse(Course course) {
        if (course.getCredits() <= 0) {
            throw new InvalidCreditsException(course.getCredits());
        }
        course.setId(nextId++);
        courses.add(course);
        return course;
    }

    public Course findById(Long id) {
        return courses.stream()
                .filter(c -> c.getId().equals(id))
                .findFirst()
                .orElseThrow(() -> new CourseNotFoundException(id));
    }

    public List<Course> getAll() {
        return courses;
    }
}
```

```java
public class CourseDemo {

    public static void main(String[] args) {
        CourseService service = new CourseService();
        service.addCourse(new Course(null, "OOP", 20));

        try {
            service.addCourse(new Course(null, "Broken", -20));
        } catch (InvalidCreditsException e) {
            System.out.println("400: " + e.getMessage());
        }

        try {
            service.findById(999L);
        } catch (CourseNotFoundException e) {
            System.out.println("404: " + e.getMessage());
        }

        System.out.println("Saved: " + service.getAll().size());
    }
}
```

```
400: Credits must be a positive number, got: -20
404: Course not found with id: 999
Saved: 1
```

---

## 11. Glossary

| Term | Plain-English meaning |
| :-- | :-- |
| **Catch** | Handle a thrown exception in a `catch` block. |
| **Checked exception** | One the compiler forces you to catch or declare (`Exception` family, except `RuntimeException`). |
| **`@ControllerAdvice`** | Spring: marks a class that handles exceptions from every controller. |
| **Custom exception** | Your own exception class, usually extending `RuntimeException`. |
| **`Error`** | Serious JVM failure. Never catch. |
| **Exception** | An object describing a problem, thrown to stop normal flow. |
| **`@ExceptionHandler`** | Spring: marks a method that handles one exception type. |
| **`finally`** | A block that always runs after `try`/`catch`. |
| **`getMessage()`** | Returns an exception's message text. |
| **HTTP status code** | A three-digit number saying how a request went: 2xx success, 4xx client error, 5xx server error. |
| **Multi-catch** | `catch (A \| B e)`: one block for several types. |
| **`orElseThrow(...)`** | `Optional` method: the value, or throw the exception the lambda builds. |
| **Propagation** | An exception travelling up through calling methods until caught. |
| **`ResponseEntity`** | Spring: a full HTTP response (status, headers, body). |
| **Re-throw** | Throwing again from a catch block. |
| **`RuntimeException`** | The parent of all unchecked exceptions. |
| **Stack trace** | The printed list of method calls showing where an exception happened. |
| **Swallowing** | Catching an exception and doing nothing. A serious bug. |
| **`throw`** | Statement that throws an exception now. |
| **`throws`** | Header clause warning a method might throw. |
| **Translating** | Catching a low-level exception and throwing a meaningful one. |
| **`try`** | Block of code that might throw. |
| **Unchecked exception** | One the compiler does not force you to handle (`RuntimeException` family). |

---

## 12. One-page cheat sheet

```java
// ---- custom exceptions: extend RuntimeException, super(message) ----
public class ThingNotFoundException extends RuntimeException {
    public ThingNotFoundException(Long id) {
        super("Thing not found with id: " + id);         // sets getMessage()
    }
}
public class InvalidThingException extends RuntimeException {
    public InvalidThingException(String message) { super(message); }
}

// ---- service: decides IF something is wrong ----
public Thing findById(Long id) {
    return things.stream()
            .filter(t -> t.getId().equals(id))
            .findFirst()
            .orElseThrow(() -> new ThingNotFoundException(id));
}
public Thing add(Thing t) {
    if (t.getName() == null || t.getName().isBlank()) {
        throw new InvalidThingException("Name is required");   // validate BEFORE saving
    }
    t.setId(nextId++);
    things.add(t);
    return t;
}

// ---- plain Java handling ----
try {
    risky();
} catch (SpecificException e) {          // most specific first
    System.out.println(e.getMessage());
} catch (OtherA | OtherB e) {            // multi-catch
    throw new MeaningfulException("..."); // translate
} finally {
    cleanUp();                           // always runs
}

// ---- Spring handler (exam Task 3) ----
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ThingNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ThingNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse(ex.getMessage(), 404));
    }
}
```

| Remember | |
| :-- | :-- |
| Exceptions are objects in a hierarchy | A parent `catch` catches child types too |
| Own exceptions extend `RuntimeException` | Unchecked: no `throws` needed |
| `super(message)` in the constructor | Otherwise the message is `null` |
| Service throws; handler reports; controller does neither | No try/catch in the controller |
| Validate before saving | Then nothing invalid is stored |
| Catch only where you can act | Never swallow, never catch `Exception` broadly |
| `finally` always runs | Never `return` inside it |
| 200 GET, 201 POST, 204 DELETE | 400 invalid, 404 missing, 409 conflict, 500 your bug |

---

## 13. Am I ready for Phase 7?

- [ ] I can explain what an exception is and how it travels up. (§2, §5.1)
- [ ] I can draw the hierarchy: `Throwable`, `Error`, `Exception`, `RuntimeException`. (§3)
- [ ] I can explain checked versus unchecked, and which my own exceptions should be. (§3.1)
- [ ] I can write `try` / `catch` / `finally`, in the right catch order. (§4)
- [ ] I can predict what `finally` does with a `return`. (§4.3)
- [ ] I can name the three ways catching goes wrong. (§4.5)
- [ ] I can write a custom exception whose message includes the id. (§6)
- [ ] I can throw it from a service with `orElseThrow`. (§7.1)
- [ ] I can validate before saving and throw a second custom exception. (§7.2)
- [ ] I can write a `@ControllerAdvice` handler that returns 404 with an `ErrorResponse`. (§8)
- [ ] I know which status code goes with which situation. (§8.4)
- [ ] I built `CourseService` with both exceptions without copying. (§10.3)

### The question that opens Phase 7

Since Phase 3, the controller has had this field, and nobody ever wrote `new ProductService()`:

```java
@Autowired
private ProductService productService;
```

An object arrives anyway. And the service still builds its own `ArrayList` inside itself: to test it with different data, or to switch to a database, you would have to edit the service. **Who should create the objects a class depends on, and how should they be handed over?**

**Next: Phase 7, Dependencies and Layered Design.**
