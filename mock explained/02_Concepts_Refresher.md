# 02: Concepts Refresher (only what this exam uses)

Each section: **idea → this exam's code → the mistake to avoid.** If a topic feels solid, skim the code block and move on.

---

## 1. The big picture: three layers

```
HTTP request
   │
   ▼
Controller   ← receives the HTTP request, returns the response. NO logic.
   │ calls
   ▼
Service      ← ALL the logic: validation, formulas use, sorting, greedy loop, throwing exceptions
   │ calls
   ▼
Repository   ← stores missions in a Map in memory. find / exists / save / all.
   │ holds
   ▼
Model        ← DispatchMission (abstract) → GroundMission, AirMission. Plus plain data classes.
```

And on the side: **exception** package → `MissionNotFoundException`, `ErrorResponse`, `GlobalExceptionHandler` (turns exceptions into HTTP responses).

**Mental rule:** *who is allowed to know about HTTP?* Only the controller and the exception handler. The service throws plain Java exceptions and never mentions "400" or "404".

---

## 2. Inheritance and `super(...)`

`GroundMission` and `AirMission` are both kinds of `DispatchMission`. The parent holds what they **share** (id, distance, load); each child adds what is **different**.

```java
public abstract class DispatchMission {            // starter gives you this
    private final long id;
    private final int distanceKm;
    private final int loadKg;
    protected DispatchMission(long id, int distanceKm, int loadKg) { ... }
    public long getId() {...}  public int getDistanceKm() {...}  public int getLoadKg() {...}
    public abstract int energyUnits();
    public abstract int priority();
}
```

```java
public class AirMission extends DispatchMission {
    private final int windPenalty;       // extra: only air missions have it
    private final boolean critical;      // extra

    public AirMission(long id, int distanceKm, int loadKg, int windPenalty, boolean critical) {
        super(id, distanceKm, loadKg);   // MUST be first: builds the parent part
        this.windPenalty = windPenalty;  // then the child's own part
        this.critical = critical;
    }
}
```

| Fact | Why it matters here |
| :-- | :-- |
| The parent's fields are `private` | In the child you **cannot** write `distanceKm`. Write `getDistanceKm()` |
| `super(...)` must be the first statement | Otherwise compile error |
| `extends` goes on the child's class line | `public class AirMission extends DispatchMission` |
| `final` fields must be set in the constructor | Every `final` field needs a value there, or it won't compile |

**Avoid:** copying `id`/`distanceKm`/`loadKg` fields into `AirMission`. That's duplication and loses the "chain to parent" marks.

---

## 3. Abstract classes and overriding

`abstract` means "incomplete on purpose". `DispatchMission` **declares** two methods with no body:

```java
public abstract int energyUnits();
public abstract int priority();
```

Every concrete child **must** implement both, or it won't compile. This is the **contract**: any mission can answer "what's your energy?" and "what's your priority?", but each answers in its own way.

```java
@Override                                  // asks the compiler: "check I really match the parent"
public int priority() {                    // same name, same return type, same (empty) parameters
    return critical ? 5 : 2;
}
```

`@Override` isn't strictly required, but "Correctly override" in the paper means: use it, and the signature must match exactly. If you write `public long priority()` or `public int Priority()`, the compiler complains, which is exactly why you use the annotation.

You can't write `new DispatchMission(...)`. It's abstract. You can only create children.

---

## 4. Polymorphism (the idea the whole paper tests)

*"Many forms."* A variable of the **parent type** can hold **any child**; calling a method runs the **child's** version.

```java
DispatchMission a = new GroundMission(101, 12, 3, true);
DispatchMission b = new AirMission(103, 8, 1, 3, true);

a.energyUnits();   // 39: GroundMission's formula runs
b.energyUnits();   // 38: AirMission's formula runs
```

In `plan(...)` you hold a `List<DispatchMission>` containing both kinds. You write **one** loop:

```java
for (DispatchMission m : eligible) {
    int e = m.energyUnits();     // Java picks Ground or Air formula automatically
    ...
}
```

The forbidden alternative (it would lose marks, and it breaks the moment someone adds a third type):

```java
if (m instanceof AirMission) { ... } else { ... }     // ✗ "concrete subclass type checks"
```

**One-line explanation for a viva or short answer:** *"The service depends only on the abstract `DispatchMission`; each subclass supplies its own `energyUnits()` and `priority()`, and the JVM chooses the right one at runtime, so adding a new mission type needs no change in the service."*

---

## 5. Integer overflow: `long` and `Math.toIntExact`

An `int` maxes out at 2,147,483,647. One more **wraps silently to a negative**:

```java
int x = Integer.MAX_VALUE;
x + 1        // -2147483648, no error
```

The fix has two parts:

```java
long energy = (2L * getDistanceKm()) + (5L * getLoadKg());   // 1) maths in long
return Math.toIntExact(energy);                              // 2) convert; throws if it doesn't fit
```

| Piece | Why |
| :-- | :-- |
| `2L` | `L` makes the literal a `long`, so `2L * int` is done in `long`. (`2 * int` is `int` maths and overflows **before** you widen it.) |
| `Math.toIntExact(long)` | Narrow to `int`; throws `ArithmeticException` instead of wrapping |

With the paper's limits (≤ 10,000 each) the largest Air energy is `3·10000 + 8·10000 + 2·10000 = 130,000`, which fits in `int`. The paper still demands the safe pattern, so write it every time. It's the marks.

Optional extra (in `addAir`): `ArithmeticException` is **not** mapped to 400 by your handler. If you want overflow to produce 400, catch it and rethrow as `IllegalArgumentException`. Not required by the paper with these limits.

---

## 6. Collections you need

| Thing | Code |
| :-- | :-- |
| Make a list | `List<DispatchMission> eligible = new ArrayList<>();` |
| Add | `eligible.add(m);` |
| Loop | `for (DispatchMission m : repo.all()) { ... }` |
| Empty result list | `new ArrayList<>()` (not `null`, because JSON must be `[]`) |
| `Collection` vs `List` | `repo.all()` returns a `Collection` (maybe no order). Copy into an `ArrayList` before sorting |
| `Map<Long, DispatchMission>` | `map.get(id)` → mission or **`null`**; `map.containsKey(id)` → boolean |

**Why sort at all?** The repository's map is a `ConcurrentHashMap`, which gives **no guaranteed order**. That's why the paper demands a fully specified sort: *deterministic*.

---

## 7. Sorting with a `Comparator` (3 keys)

A `Comparator` says which of two items goes first. You need: **priority DESC → energy ASC → id ASC.**

### Version A: chained (shortest, easiest to read)

```java
eligible.sort(
    Comparator.comparingInt(DispatchMission::priority).reversed()   // priority DESC
              .thenComparingInt(DispatchMission::energyUnits)        // energy ASC
              .thenComparingLong(DispatchMission::getId));           // id ASC
```

- `DispatchMission::priority` is a **method reference** = `m -> m.priority()`.
- Use `comparingInt` for `int` results, `comparingLong` for `long` (id is `long`).
- **Trap:** `.reversed()` flips **everything before it**. Put it right after the first key. Never at the very end.

### Version B: manual (the model solution; works everywhere, easiest to trace)

```java
Collections.sort(eligible, new Comparator<DispatchMission>() {
    @Override
    public int compare(DispatchMission first, DispatchMission second) {
        int result = Integer.compare(second.priority(), first.priority());   // DESC: swapped!
        if (result != 0) return result;
        result = Integer.compare(first.energyUnits(), second.energyUnits()); // ASC: normal order
        if (result != 0) return result;
        return Long.compare(first.getId(), second.getId());                  // ASC
    }
});
```

**How to remember:** `compare(a, b)` returns negative if `a` goes first.
- Ascending → `Integer.compare(a.x, b.x)` (a smaller → a first).
- Descending → swap: `Integer.compare(b.x, a.x)`.
- Only continue to the next key when the previous says "equal" (`result == 0`).

---

## 8. The greedy algorithm

**Greedy = at each step make the best-looking choice right now, never reconsider.** Here "best" is already decided by the sort, so the algorithm is just:

```
remaining = capacity
for each mission in sorted order:
    if mission.energy <= remaining:   take it, remaining -= energy
    else:                             put its id in skipped (and keep going)
```

In code with a running total:

```java
int used = 0;
for (DispatchMission m : eligible) {
    int e = m.energyUnits();
    if (e <= capacity - used) {            // "fits the remaining capacity" (≤, not <)
        selected.add(new PlanLine(m.getId(), m.getClass().getSimpleName(), m.priority(), e));
        used += e;
    } else {
        skipped.add(m.getId());
    }
}
int remaining = capacity - used;
```

- `<=` matters: a mission of exactly the remaining energy **fits** (that's how 125 → remaining 0 works).
- `capacity - used` is the "remaining capacity" test; `used + e <= capacity` is equivalent.
- `m.getClass().getSimpleName()` = the class's short name: `"AirMission"`. It labels the type without `instanceof`.

---

## 9. Exceptions

An **exception** = "something went wrong, stop this method and jump to whoever handles it."

```java
public class MissionNotFoundException extends RuntimeException {      // starter gives this
    public MissionNotFoundException(long id) { super("Mission not found: " + id); }
}
```

- `extends RuntimeException` = **unchecked**: no `throws` clauses needed.
- `super(message)` sets what `getMessage()` returns. That's the "descriptive message".
- **Throw:** `throw new MissionNotFoundException(id);`
- **Built-in** for bad input: `throw new IllegalArgumentException("distanceKm must be between 1 and 10000.");`

**The flow:**

```
Service throws ──► (nobody catches it in the service/controller) ──► Spring finds
@ExceptionHandler in GlobalExceptionHandler ──► builds ResponseEntity(status + ErrorResponse JSON)
```

---

## 10. `GlobalExceptionHandler` and HTTP status codes

```java
@RestControllerAdvice      // "applies to every controller; return values become JSON"
public class GlobalExceptionHandler {

    @ExceptionHandler(MissionNotFoundException.class)    // "when THIS is thrown, run me"
    public ResponseEntity<ErrorResponse> handleNotFound(MissionNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)                      // 404
                .body(new ErrorResponse("NOT_FOUND", ex.getMessage()));
    }
    ...
}
```

| Situation | Status | `code` |
| :-- | :-- | :-- |
| Success, read | 200 OK (default) | - |
| Success, created | **201 Created** | - |
| Bad input (range, duplicate, capacity ≤ 0) | **400** Bad Request | `BAD_REQUEST` |
| Unknown mission ID | **404** Not Found | `NOT_FOUND` |

Response body JSON looks like `{"code":"NOT_FOUND","message":"Mission not found: 999"}`. Jackson builds it from `ErrorResponse`'s getters.

---

## 11. Spring wiring: annotations and DI

| Annotation | Meaning | Where |
| :-- | :-- | :-- |
| `@SpringBootApplication` | Entry point; scans this package and below | `DispatchApplication` (base package: **keep it there**) |
| `@RestController` | Class handles HTTP, returns JSON | controller |
| `@RequestMapping("/missions")` | Base path for all methods in the class | controller class |
| `@GetMapping("/x")`, `@PostMapping("/x")` | Method handles that verb + sub-path | controller methods |
| `@PathVariable("id")` | Value taken from `{id}` in the URL | parameter |
| `@RequestParam("capacity")` | Value from `?capacity=60` | parameter |
| `@RequestParam(name="urgentOnly", defaultValue="false")` | Optional, with default | parameter |
| `@RequestBody` | Parse the JSON body into this object | parameter |
| `@ResponseStatus(HttpStatus.CREATED)` | Use 201 instead of 200 | method |
| `@Service` / `@Repository` | Spring creates and manages one instance | classes |
| `@RestControllerAdvice` / `@ExceptionHandler` | global error handling | exception package |

**Dependency injection (constructor injection):** the starter already has

```java
private final DispatchService service;
public DispatchController(DispatchService service) { this.service = service; }
```

You never write `new DispatchService()`. Spring sees the constructor needs a `DispatchService` and passes the one it manages (that is **Inversion of Control**). Same for `DispatchService(MissionRepository repo)`. **Don't change these.** Only use `service` / `repo`.

**Why not `new`?** Then you'd have *two* repositories: one holding mission 104 and one not. Spring gives everyone the same single instance.

---

## 12. JSON and Jackson (why classes and getters matter, and records are banned)

- Response: Jackson looks at **getters** (`getId()`, `isCritical()`) and writes one JSON field per getter. So `GET /missions` shows `id`, `distanceKm`, `loadKg`, `windPenalty`, `critical` for air.
- **`energyUnits()` and `priority()` are NOT in the `/missions` JSON** because they don't start with `get`/`is`. That's expected. You see energy via `/missions/{id}/energy` or the plan.
- Request: for `@RequestBody NewAirMission`, Jackson uses the **no-arg constructor + setters** (`setId`, `setDistanceKm`, ...). That's why the supplied class has both. Don't delete them.
- Plain classes vs records: records use accessors `x()`, classes use `getX()`. This paper says **classes**.

```json
POST body →  {"id":104,"distanceKm":6,"loadKg":1,"windPenalty":2,"critical":false}
```

---

## 13. Debugging reflexes

| Symptom | Likely cause |
| :-- | :-- |
| App won't start, "UnsupportedOperationException" in a stack trace | A starter TODO still throws; you're calling an unfinished method. Fill it |
| `cannot find symbol AirMission` | You haven't created it yet, or wrong package / not saved |
| 404 on a URL you wrote correctly | Wrong path in annotation (doubled `/missions`), or app not restarted |
| 500 Internal Server Error | An exception nobody handled: read the red stack trace's top lines |
| 400 from Spring with HTML/odd body on `capacity=abc` | Type mismatch handled by Spring itself, not your code. Ignore |
| Plan output order looks wrong | Comparator: DESC swap, or `.reversed()` placement |
| 415 Unsupported Media Type on POST | Missing header `Content-Type: application/json` |
| Change not showing | **Restart** the app. In-memory data resets on restart too |
