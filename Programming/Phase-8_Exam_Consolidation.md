# Phase 8: Exam Consolidation

**Module:** 5COSC019W Object Oriented Programming
**Before this:** Phases 0 to 7.
**Goal:** walk into the lab practical knowing its shape, a minute-by-minute plan, the exact code patterns for each task, how to check your own work, and how to submit safely.
**Comes with:** `Practice_Tickets_STARTER.zip` (a practice paper in the mock's shape) and `Practice_Tickets_SOLUTION.zip` (the model answer).

**Built from your files:** Student Assessment Briefing, Lecture 6 Section 07 (Mock Lab Exam Briefing), and every earlier lecture and tutorial.

> **Important.** The practice paper here is **mine**, written in the shape your lecturer described for the mock. It is not the real exam, not the mock, and not one of the three practice papers your briefing mentions (parking, dispatch, scheduler). Follow your official instructions for the real sitting wherever they differ from anything here.

---

## Where we are on the roadmap

| Phase | Title | Status |
| :-- | :-- | :-- |
| Pre-Phase 0 to Phase 7 | Java, classes, encapsulation, collections, inheritance, polymorphism, exceptions, layers | Done |
| **Phase 8** | **Exam Consolidation** | **You are here** |
| Spring track | Your other chat | Next |

---

## 1. The exam at a glance

### 1.1 Format (from your briefing)

| Item | Detail |
| :-- | :-- |
| Type | Individual, timed, in-lab programming assessment in IntelliJ |
| Project | A supplied Java / Maven / Spring Boot starter project |
| Weight | 40% of the module (qualifying mark 30, per Lecture 1) |
| Working time | **100 minutes** |
| Section A | 4 source-code tasks, **80 marks**, read by a marker, **partial credit possible** |
| Section B | Functional checks of your **running** program, **20 marks** |
| Submission | One ZIP of the complete Maven project, including `pom.xml` and `src/main` |

### 1.2 The marks, and which phase teaches each

| Task | Competency (briefing) | Marks | What it looked like in the mock (Lecture 6) | Study |
| :-- | :-- | :-- | :-- | :-- |
| A1 | Java OOP implementation | **18** | "The file exists. Add a field, a constructor calling `super(...)`, and its summary." | Phases 1, 2, 4 |
| A2 | Applied OOP and polymorphism | **22** | "You make the file this time: a field, two constructors and its own `getSummary()`." | Phases 4, 5 |
| A3 | Services, validation and exceptions | **20** | "A custom exception thrown in the service, turned into 404 by a global handler." | Phases 3, 6 |
| A4 | Spring REST API integration | **20** | "GET by id, and POST returning 201 Created. No try/catch in the controller." | Spring track, Phase 6 §8 |
| B | Functional verification | **20** | Six checks on the running program (Section 1.3) | All |

The mock also had a Task 5 (*"a second exception and handler, returning 400"*) described as *"the extra step that makes the mock slightly harder, on purpose"*. Validation with a 400 still appears in Section B, so prepare it.

### 1.3 Section B: the six checks (from the mock briefing)

| # | Request | Must return |
| :-- | :-- | :-- |
| 1 | GET all | **200** with the seeded items, and the two subclasses' summaries **read differently** |
| 2 | GET by an existing id | **200** with that one item |
| 3 | GET by an unknown id | **404**: not 500, and not an empty 200 |
| 4 | (same request) | The 404 body is **JSON** with a message that **names the id** |
| 5 | POST valid | **201 Created**, with the saved item **and its new id** |
| 6 | POST invalid | **400 Bad Request** with a JSON message, and **nothing is saved** |

**Section B only runs if your ZIP starts.** A project that does not compile scores zero here, however good the code.

### 1.4 Rules

From the briefing: individual work; no collaboration or communication; follow the **approved** policy on AI tools, documentation and notes for the actual sitting. For the mock, Lecture 6 stated: *open book: slides, Blackboard materials and the Java and Spring docs; no AI assistants.* Talk to the invigilator about **tools** if something will not run, never about the logic. Check the final instructions on the day.

---

## 2. Minute-by-minute plan

Based on Lecture 6's mock timeline, scaled to 100 minutes.

| Clock | Do |
| :-- | :-- |
| **0:00–0:05** | Download, unzip, open the folder **that contains `pom.xml`**. Wait for Maven. Press Run. Open Swagger UI (`/swagger-ui.html`). Do any "Task 0" (e.g. sign the README). |
| **0:05–0:12** | **Read everything before coding** (Section 3). Write the parent's constructor parameter order on paper. |
| **0:12–0:27** | **Task 1.** Restart and check GET all. |
| **0:27–0:45** | **Task 2.** Restart and check GET all: two different summaries. |
| **0:45–0:58** | **Task 3.** Restart and test the unknown id gives 404. |
| **~0:58** | **ZIP and upload a safety copy.** (Lecture 6: *"Upload a safety copy after Task 3."*) |
| **0:58–1:15** | **Task 4.** Restart and test GET by id and POST. |
| **1:15–1:28** | Validation / second handler if asked; any stretch. |
| **1:28–1:33** | **Stop coding.** Restart. Run all six Section B checks. |
| **1:33–1:37** | ZIP the project. Open the ZIP to check it. |
| **1:37–1:40** | Upload, replace the safety copy, press **Submit**, wait for confirmation. |

The briefing: *reserve roughly the final 10 minutes for final checks, verifying the ZIP and uploading it.*

**If you are stuck on a task for more than 5 minutes, write a sensible partial attempt that compiles and move on.** Lecture 6: *"An empty TODO earns nothing; a sensible attempt earns partial marks."*

---

## 3. Reading the starter (the first seven minutes)

Lecture 6 says the starter is a running project where *the abstract parent class, the repository and `ErrorResponse`* are provided (read, do not edit), *constructor injection in the service and controller* is provided, and *every file you complete already exists, marked `// TODO`* (except the one new subclass).

Read in this order, and write the answers down:

| File | Questions to answer |
| :-- | :-- |
| Abstract parent (`model/`) | Field names and types? **Constructor parameter order?** Which methods are `abstract` (you must override every one)? Are fields `private` (use getters) or `protected`? Setters present? |
| Existing subclass | What is already there? What do the `// TODO`s ask for? |
| Repository interface | Exact method names and return types: `findById` returns `Optional`? `save` returns the saved object? |
| Repository class | Is data seeded? Does `save` assign ids? |
| `ErrorResponse` | **Constructor parameters**, in order. |
| Service | What is injected? Where are the `// TODO`s? |
| Controller | Base path (`@RequestMapping`)? Which endpoints exist? |
| Exception package | Which classes exist? Is there a handler class already? |

Also press **Ctrl + Shift + F** in IntelliJ and search for `TODO` to list every task location at once.

---

## 4. The practice paper: Concert Tickets

Open `Practice_Tickets_STARTER.zip`, unzip it, and open the `ticket-practice` folder in IntelliJ. Its `README.md` has the full task list. **Set a 100-minute timer and do it before reading Section 5.**

### 4.1 What is provided

**`Ticket.java`** (abstract parent):

```java
public abstract class Ticket {

    private Long id;
    private String eventName;
    private double basePrice;

    public Ticket() {}

    public Ticket(Long id, String eventName, double basePrice) { ... }

    // getters and setters for id, eventName, basePrice

    public abstract double calculatePrice();

    public abstract String getSummary();
}
```

From the read-through: constructor order is **(id, eventName, basePrice)**; fields are **private** so subclasses must use `getEventName()` / `getBasePrice()`; there are **two** abstract methods to override in every subclass.

**`TicketRepository`**: `findAll()`, `findById(Long)` returning `Optional<Ticket>`, `save(Ticket)`, `deleteById(Long)`.

**`ErrorResponse`**: constructor `(String message, int status)`.

**Service and controller**: constructor injection already written; GET all already works.

### 4.2 The tasks (summary)

| Task | Marks | Ask |
| :-- | :-- | :-- |
| 1 | 18 | Complete `StandardTicket`: field `seatRow`; full constructor with `super(...)`; `calculatePrice()` = base price; `getSummary()` = `"Standard: <event> \| Row <row> \| £<price>"`. |
| 2 | 22 | Create `VipTicket`: field `backstagePass` (boolean); two constructors; price = base × 1.5, + 50 with a backstage pass; summary `"VIP: <event> \| Backstage: yes/no \| £<price>"`. |
| 3 | 20 | `TicketNotFoundException`; service throws it; handler returns 404 with `ErrorResponse`. |
| 4 | 20 | `GET /tickets/{id}` and `POST /tickets/vip` returning 201. No try/catch in the controller. |
| 5 | bonus | `InvalidTicketException` for blank name or price ≤ 0; second handler returning 400; nothing saved. |

---

## 5. Model answer, task by task

Everything below compiles. It was tested against stand-in Spring types plus a plain-Java run of all six Section B scenarios (Spring itself could not be downloaded in my workspace, so do run the full project yourself in IntelliJ).

### 5.1 Task 1: complete the existing subclass

```java
public class StandardTicket extends Ticket {

    private String seatRow;

    public StandardTicket() {}

    public StandardTicket(Long id, String eventName, double basePrice, String seatRow) {
        super(id, eventName, basePrice);
        this.seatRow = seatRow;
    }

    public String getSeatRow() { return seatRow; }
    public void setSeatRow(String seatRow) { this.seatRow = seatRow; }

    @Override
    public double calculatePrice() {
        return getBasePrice();
    }

    @Override
    public String getSummary() {
        return "Standard: " + getEventName() + " | Row " + seatRow + " | £" + calculatePrice();
    }
}
```

| Marking point (likely) | Where |
| :-- | :-- |
| Private field of the right type | `private String seatRow;` |
| Constructor with parent values first | `(Long id, String eventName, double basePrice, String seatRow)` |
| `super(...)` first, in the parent's order | `super(id, eventName, basePrice);` |
| Own field assigned with `this.` | `this.seatRow = seatRow;` |
| Getter/setter following JavaBean naming | `getSeatRow`, `setSeatRow` |
| Every abstract method overridden with `@Override` | both methods |
| Parent's private fields reached through getters | `getEventName()`, `getBasePrice()` |
| Summary format exactly as specified | spaces, `\|` and `£` as asked |

`getSummary()` calls `calculatePrice()` rather than repeating the formula: one rule, one place.

### 5.2 Task 2: create a new subclass from nothing

In IntelliJ: right-click the `model` package → New → Java Class → `VipTicket`. The `package` line is added for you.

```java
public class VipTicket extends Ticket {

    public static final double VIP_MULTIPLIER = 1.5;
    public static final double BACKSTAGE_FEE = 50.0;

    private boolean backstagePass;

    public VipTicket() {}

    public VipTicket(Long id, String eventName, double basePrice, boolean backstagePass) {
        super(id, eventName, basePrice);
        this.backstagePass = backstagePass;
    }

    public boolean isBackstagePass() { return backstagePass; }
    public void setBackstagePass(boolean backstagePass) { this.backstagePass = backstagePass; }

    @Override
    public double calculatePrice() {
        double price = getBasePrice() * VIP_MULTIPLIER;
        if (backstagePass) {
            price += BACKSTAGE_FEE;
        }
        return price;
    }

    @Override
    public String getSummary() {
        String backstage = "no";
        if (backstagePass) {
            backstage = "yes";
        }
        return "VIP: " + getEventName() + " | Backstage: " + backstage + " | £" + calculatePrice();
    }
}
```

| Point | Why it matters |
| :-- | :-- |
| `extends Ticket` | IS-A: it can go in the `List<Ticket>`. |
| **Two** constructors | The no-arg one is what Spring's Jackson uses to build a `VipTicket` from the POST body. Without it, POST fails. |
| Setter for the new field | Jackson fills fields through setters after the no-arg constructor. |
| `isBackstagePass()` | Boolean getter uses `is`. |
| Constants for 1.5 and 50.0 | Named numbers (Phase 2). Not required, but clean. |
| A **different** summary from `StandardTicket` | Section B check 1: *"summaries read differently"*. |

Then uncomment the VIP seed line. Restart. `GET /tickets` should now show:

```
1  Standard: Rock Night | Row F | £40.0
2  VIP: Jazz Evening | Backstage: yes | £140.0
```

(60 × 1.5 = 90, + 50 = 140.) That is polymorphism in one request: the same `getSummary()` call, two answers.

### 5.3 Task 3: not found becomes 404

**New file** `exception/TicketNotFoundException.java`:

```java
public class TicketNotFoundException extends RuntimeException {

    public TicketNotFoundException(Long id) {
        super("Ticket not found with id: " + id);
    }
}
```

**Service** method:

```java
public Ticket getTicketById(Long id) {
    return repository.findById(id)
            .orElseThrow(() -> new TicketNotFoundException(id));
}
```

**Handler** (in the existing `GlobalExceptionHandler`, which already has `@ControllerAdvice`):

```java
@ExceptionHandler(TicketNotFoundException.class)
public ResponseEntity<ErrorResponse> handleNotFound(TicketNotFoundException ex) {
    return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse(ex.getMessage(), 404));
}
```

Imports needed in the handler (Alt + Enter adds them):

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
```

| Point | Why |
| :-- | :-- |
| `extends RuntimeException` | Unchecked: no `throws` anywhere. |
| `super("... " + id)` | Sets `getMessage()`. Without it the 404 body has no message, and check 4 fails. |
| Throw in the **service** | The service owns the rule "a ticket must exist". |
| `HttpStatus.NOT_FOUND` and `404` | Status line and body agree. |
| `@ControllerAdvice` present | Without it the handler never runs and you get 500. |

**Now upload your safety copy.**

### 5.4 Task 4: the two endpoints

**Service** method for POST:

```java
public Ticket createVipTicket(VipTicket ticket) {
    ticket.setId(null);
    return repository.save(ticket);
}
```

`setId(null)` ignores any id the client tried to send, so the repository always assigns a fresh one.

**Controller** methods:

```java
@GetMapping("/{id}")
public Ticket getById(@PathVariable Long id) {
    return service.getTicketById(id);
}

@PostMapping("/vip")
@ResponseStatus(HttpStatus.CREATED)
public Ticket createVip(@RequestBody VipTicket ticket) {
    return service.createVipTicket(ticket);
}
```

| Piece | Meaning |
| :-- | :-- |
| `@GetMapping("/{id}")` | Handles `GET /tickets/{id}` (the class already has `@RequestMapping("/tickets")`). `{id}` is a slot in the URL. |
| `@PathVariable Long id` | Copies the slot's value into `id`. |
| `@PostMapping("/vip")` | Handles `POST /tickets/vip`. |
| `@RequestBody VipTicket ticket` | Jackson builds a `VipTicket` from the JSON body (no-arg constructor, then setters). |
| `@ResponseStatus(HttpStatus.CREATED)` | 201 instead of the default 200. Forgetting it is on Lecture 6's list of where people lose marks. |
| **No try/catch** | An unknown id throws in the service; the handler turns it into 404 by itself. |

Why POST takes `VipTicket`, not `Ticket`: `Ticket` is abstract, so Jackson cannot build one. A concrete subclass can be built.

### 5.5 Task 5: invalid data becomes 400

**New file** `exception/InvalidTicketException.java`:

```java
public class InvalidTicketException extends RuntimeException {

    public InvalidTicketException(String message) {
        super(message);
    }
}
```

**Service**, validation **before** saving:

```java
public Ticket createVipTicket(VipTicket ticket) {
    if (ticket.getEventName() == null || ticket.getEventName().isBlank()) {
        throw new InvalidTicketException("Event name is required");
    }
    if (ticket.getBasePrice() <= 0) {
        throw new InvalidTicketException("Base price must be greater than zero");
    }
    ticket.setId(null);
    return repository.save(ticket);
}
```

**Second handler**:

```java
@ExceptionHandler(InvalidTicketException.class)
public ResponseEntity<ErrorResponse> handleInvalid(InvalidTicketException ex) {
    return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse(ex.getMessage(), 400));
}
```

Because the checks come before `repository.save`, an invalid ticket is never stored: check 6.

### 5.6 Stretch: DELETE

```java
// service
public void deleteTicket(Long id) {
    getTicketById(id);            // throws TicketNotFoundException if missing -> 404
    repository.deleteById(id);
}

// controller
@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void delete(@PathVariable Long id) {
    service.deleteTicket(id);
}
```

### 5.7 Section B on the practice paper

Test in Swagger UI (`/swagger-ui.html`, if the project includes springdoc) or the browser for GETs. The expected results, verified by the plain-Java run:

| # | Request | Expected |
| :-- | :-- | :-- |
| 1 | `GET /tickets` | 200; ids 1 and 2; one `Standard: ...` and one `VIP: ...` summary |
| 2 | `GET /tickets/2` | 200; the VIP ticket |
| 3 | `GET /tickets/999` | 404 |
| 4 | (same) | `{"message":"Ticket not found with id: 999","status":404,"timestamp":"..."}` |
| 5 | `POST /tickets/vip` with `{"eventName":"Opera Gala","basePrice":80,"backstagePass":false}` | 201; `"id":3`; summary `VIP: Opera Gala \| Backstage: no \| £120.0` |
| 6 | `POST /tickets/vip` with `{"eventName":"Broken","basePrice":-10}` | 400; message `Base price must be greater than zero`; `GET /tickets` unchanged |

You will notice the JSON for each ticket also contains a `"summary"` field. That is Jackson treating `getSummary()` as a getter (Phase 2, Section 9): any public `getX()` becomes a JSON field.

> If the practice project does not build on your machine because of the Spring Boot version in its `pom.xml`, create a fresh project at start.spring.io exactly as in Lecture 1 (Spring Web dependency), then copy the `src/main/java` folders across.

---

## 6. Checklists to run before moving on from each task

### Task 1 / Task 2 (subclasses)

- [ ] `extends` the right parent; new fields only, `private`
- [ ] No-arg constructor `public X() {}`
- [ ] Full constructor: parent values first, in the **parent's order**
- [ ] `super(...)` is the **first** line
- [ ] `this.field = field;` for each new field
- [ ] Getter and setter for each new field (`is` for booleans)
- [ ] **Every** abstract method overridden, each with `@Override`, exact spelling and parameters
- [ ] Parent's private fields used through getters
- [ ] Output text matches the specification character for character
- [ ] Seed data uncommented if instructed; restart; GET all shows both summaries, different

### Task 3 (exceptions)

- [ ] `extends RuntimeException`
- [ ] Constructor calls `super("... " + id)`
- [ ] Thrown in the **service** with `orElseThrow(() -> new X(id))`
- [ ] Handler class has `@ControllerAdvice`
- [ ] Handler method has `@ExceptionHandler(X.class)` and parameter `X ex`
- [ ] Returns `ResponseEntity.status(HttpStatus.NOT_FOUND).body(new ErrorResponse(ex.getMessage(), 404))`
- [ ] Restart; unknown id gives 404 and JSON naming the id
- [ ] **Safety copy uploaded**

### Task 4 (endpoints)

- [ ] `@GetMapping("/{id}")` with `@PathVariable Long id`
- [ ] `@PostMapping` with `@RequestBody` of a **concrete** class
- [ ] `@ResponseStatus(HttpStatus.CREATED)` on POST
- [ ] Each method is one line that calls the service
- [ ] No try/catch anywhere in the controller
- [ ] New paths do not clash with existing ones (a fixed path like `/search` must not be confused with `/{id}`)

### Validation (400)

- [ ] Checks happen **before** `save`
- [ ] Null **and** blank checked for text (`== null || isBlank()`)
- [ ] Second exception class, second handler, `HttpStatus.BAD_REQUEST`, `400`
- [ ] After an invalid POST, GET all shows nothing new

---

## 7. Error lookup: symptom to fix

### Compile-time (red in IntelliJ)

| Message | Fix |
| :-- | :-- |
| `cannot find symbol` | Spelling, capitals, missing import (**Alt + Enter**), or calling a child method through a parent variable. |
| `X is not abstract and does not override abstract method m() in Parent` | Override every abstract method, with exactly the same header. |
| `method does not override or implement a method from a supertype` | Your `@Override` method's name or parameters do not match the parent. |
| `constructor Parent in class Parent cannot be applied to given types` | Your `super(...)` arguments are missing, in the wrong order, or the wrong types. |
| `call to super must be first statement in constructor` | Move `super(...)` to line one. |
| `x has private access in Parent` | Use the getter. |
| `incompatible types: int cannot be converted to Long` | Write `1L`, not `1`. |
| `Parent is abstract; cannot be instantiated` | Create a concrete subclass instead. |
| `missing return statement` | Every path of a non-void method must `return`. |
| `unreported exception ... must be caught or declared` | Your exception extends `Exception`; change it to `RuntimeException`. |

### Startup (red text in the console, app does not start)

| Message | Fix |
| :-- | :-- |
| `Port 8080 was already in use` | Stop the old run (red square), then run again. |
| `required a bean of type '...' that could not be found` | Missing `@Service` / `@Repository`, or the class is outside the main package. |
| `required a single bean, but 2 were found` | Two implementations of one interface; delete or mark one `@Primary`. |

### Runtime (wrong response)

| Symptom | Fix |
| :-- | :-- |
| 500 instead of 404 | Handler missing `@ControllerAdvice`, wrong exception type in `@ExceptionHandler`, or the service throws something else (e.g. plain `orElseThrow()`). |
| 404 body has `"message": null` | Custom exception constructor forgot `super(message)`. |
| 200 with empty body for an unknown id | Service returns `null` / `.orElse(null)`. Throw instead. |
| POST returns 200 | Add `@ResponseStatus(HttpStatus.CREATED)`. |
| POST returns 400 for valid JSON / cannot construct | Subclass missing its no-arg constructor or setters; or `@RequestBody` uses the abstract parent. |
| A field missing from the JSON | Getter missing or misspelled (`getname`). |
| Both summaries identical | Override missing or misspelled; check `@Override`. |
| Invalid POST still saved | Validation is after `save`. Move it before. |
| New endpoint not in Swagger | You did not restart. |
| `NullPointerException` in a filter | Something in the list has a `null` id. |

---

## 8. Submitting: one ZIP, done right

From Lecture 6:

1. In IntelliJ, right-click the project folder → **Open In → Explorer**.
2. Go **up one folder**. Right-click the project folder → **Compress to → ZIP file**.
3. Rename it as instructed (the mock used `Mock_<student ID>.zip`).
4. **Open the ZIP to check**: it contains the project folder, with `pom.xml` and `src` inside, and your latest changes.
5. Upload **the ZIP itself**, not a folder, not a single `.java` file.
6. Replace the safety copy. Press **Submit**. Wait for confirmation before the timer ends.

Lecture 6: *"Your marker unzips exactly what you uploaded and runs it. A zip of `src` alone, or of the untouched starter, cannot score Section B."*

**Practise this now with the practice paper.** It takes two minutes once you have done it, and ten when you are panicking.

---

## 9. Where people lose marks (Lecture 6, plus the guides)

| Mistake | Cost |
| :-- | :-- |
| Forgetting `@Override` (and misspelling the method) | The parent's version runs; summaries come out wrong; nothing warns you. |
| Returning 200 from POST | Check 5 fails. |
| try/catch in the controller | Rule in the wrong layer; your handler never runs. |
| No `@ControllerAdvice` | Handler never registered; 500 comes back. |
| Exception without `super(message)` | Check 4 fails. |
| Validation after saving | Check 6 fails. |
| Missing no-arg constructor on the new subclass | POST cannot build the object. |
| Zipping the wrong thing | Section B scores zero. |
| Uploading only at the end | No recovery if anything goes wrong. |
| Leaving a TODO empty | Zero for that part. A compiling partial attempt earns partial credit. |

---

## 10. Mixed self-test (all phases)

Answer aloud, then check.

| # | Question | Answer |
| :-- | :-- | :-- |
| 1 | What does `7 / 2` give, and why? | `3`: integer division drops the decimal. |
| 2 | What is the difference between a class and an object? | A class is a blueprint/type; an object is one instance built from it with `new`. |
| 3 | What does `this.name = name;` do? | Stores the parameter in the current object's field. |
| 4 | Why are fields `private`? | Encapsulation: only the class's own methods, which can validate, may change them. |
| 5 | Getter name for `private boolean active`? | `isActive()`. |
| 6 | Why does a Spring model class need a no-arg constructor? | Jackson builds the object empty, then calls setters. |
| 7 | `List` vs `ArrayList`? | `List` is the interface (contract); `ArrayList` the class implementing it. |
| 8 | How do you compare two `Long` ids? | `a.equals(b)`, never `==`. |
| 9 | What does `findFirst()` return? | An `Optional`. |
| 10 | What does a subclass never inherit? | Its parent's constructors. It calls them with `super(...)`. |
| 11 | Where must `super(...)` go? | The first line of the child constructor. |
| 12 | IS-A or HAS-A: `Order` and `Customer`? | HAS-A: use a field. |
| 13 | `Parent p = new Child();` Which version of an overridden method runs? | `Child`'s. The object decides. |
| 14 | Can you call a child-only method through `p`? | No. The reference type decides what is allowed. |
| 15 | What does `@Override` protect against? | A misspelled or mismatched override compiling silently. |
| 16 | Can you write `new` on an abstract class? | No. |
| 17 | Abstract class or interface for shared fields? | Abstract class. |
| 18 | Checked or unchecked for your own exceptions? | Unchecked: `extends RuntimeException`. |
| 19 | Which layer throws `XNotFoundException`? | The service. |
| 20 | Which layer turns it into 404? | The `@ControllerAdvice` handler. |
| 21 | Status codes for successful POST and DELETE? | 201 and 204. |
| 22 | Status for invalid input? For missing resource? | 400. 404. |
| 23 | Why constructor injection? | `final`, explicit, testable, fails at startup. |
| 24 | Where does `@Repository` go? | On the implementing class, not the interface. |
| 25 | Name the four OOP pillars. | Encapsulation, inheritance, polymorphism, abstraction. |

---

## 11. A revision plan for the time left

It is Friday evening. Assuming the exam is on Monday, here is one way to use the time. Adjust to your actual exam slot.

| When | Do | Guide |
| :-- | :-- | :-- |
| **Fri evening** | Pre-Phase 0 (type every example) and Phase 1 | Pre-0, 1 |
| **Sat morning** | Phase 2 and Phase 3 | 2, 3 |
| **Sat afternoon** | Phase 4 and Phase 5. **Highest exam weight (40 marks).** Do `Vehicle` and `Shape` from scratch. | 4, 5 |
| **Sat evening** | Phase 6 and Phase 7 | 6, 7 |
| **Sun morning** | **Spring track** in your other chat: running the project, `@RestController`, `@GetMapping`/`@PostMapping`, `@PathVariable`, `@RequestBody`, `@ResponseStatus`, `ResponseEntity`, Swagger, `@ControllerAdvice` | Spring chat |
| **Sun afternoon** | **Timed practice paper**: 100 minutes, starter ZIP, no peeking. Then compare with the solution. Practise zipping. | Phase 8 |
| **Sun evening** | Your lecturer's mock and practice papers (parking, dispatch, scheduler), if released | Your files |
| **Exam morning** | Re-do the practice paper from the blank starter in 60 minutes. Read the cheat sheets. | All cheat sheets |

If time is short, protect this order: **Phase 4 → Phase 5 → Phase 6 → Spring track → practice paper**. Those cover all four Section A tasks.

---

## 12. The one-page exam sheet

```java
// ======== TASK 1/2: a subclass ========
public class Child extends Parent {
    private Type extra;                                         // new field(s) only
    public Child() {}                                           // for Jackson (POST)
    public Child(Long id, String name, double base, Type extra) {
        super(id, name, base);                                  // FIRST, parent's order
        this.extra = extra;
    }
    public Type getExtra() { return extra; }                    // isExtra() for boolean
    public void setExtra(Type extra) { this.extra = extra; }
    @Override public double calculateSomething() { return getBase() * 1.5; }   // getters!
    @Override public String getSummary() { return "Child: " + getName() + " | " + extra; }
}

// ======== TASK 3: not found -> 404 ========
public class ThingNotFoundException extends RuntimeException {
    public ThingNotFoundException(Long id) { super("Thing not found with id: " + id); }
}
// service
public Thing getById(Long id) {
    return repository.findById(id).orElseThrow(() -> new ThingNotFoundException(id));
}
// handler
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ThingNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ThingNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse(ex.getMessage(), 404));
    }
    @ExceptionHandler(InvalidThingException.class)                 // 400 version
    public ResponseEntity<ErrorResponse> handleInvalid(InvalidThingException ex) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                .body(new ErrorResponse(ex.getMessage(), 400));
    }
}

// ======== validation: BEFORE saving ========
public Thing create(Child t) {
    if (t.getName() == null || t.getName().isBlank()) throw new InvalidThingException("Name is required");
    if (t.getBase() <= 0) throw new InvalidThingException("Base must be greater than zero");
    t.setId(null);
    return repository.save(t);
}

// ======== TASK 4: endpoints ========
@GetMapping("/{id}")
public Thing getById(@PathVariable Long id) { return service.getById(id); }

@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public Thing create(@RequestBody Child t) { return service.create(t); }

@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void delete(@PathVariable Long id) { service.delete(id); }
```

| Before you upload | |
| :-- | :-- |
| Restarted after the last change | All six Section B checks pass |
| ZIP contains the project folder with `pom.xml` and `src` | Latest changes are inside |
| Uploaded the ZIP itself | Pressed Submit and saw confirmation |

---

## 13. Ready?

- [ ] I know the marks for each task and what each looked like in the mock. (§1)
- [ ] I have a minute-by-minute plan, including the safety copy. (§2)
- [ ] I can read a starter project and answer the questions in §3 in seven minutes.
- [ ] I have done the practice paper in 100 minutes without looking. (§4)
- [ ] I have compared my answer with the model answer and understood every difference. (§5)
- [ ] I can run all six Section B checks myself. (§5.7)
- [ ] I know the fix for each common error. (§7)
- [ ] I have practised zipping and checking the ZIP. (§8)
- [ ] I scored at least 20 of 25 on the mixed self-test. (§10)

**Next: the Spring track in your other chat.** Bring Lecture 1 §04–07, Tutorial 1, and the Spring halves of Tutorials 2–5, Lecture 6 §05–06 and Lecture 7 §05.

Good luck.
