# S6: Exam Toolkit and Full Solutions to the Three Practice Papers

**Before this:** S1 to S5 and OOP Phases 0 to 8.
**Goal:** know every Java technique the practice papers use, have a 100-minute plan, and see each paper solved task by task with the reasoning, then check your own attempt against the six Section B behaviours.
**Comes with:** `SIMPLE_Parking_SOLUTION.zip`, `MODERATE_Dispatch_SOLUTION.zip`, `EXTREME_Scheduler_SOLUTION.zip` (complete projects, each with a `section-b-checks.http` request file).

**Built from:** your practice pack (`STUDENT_RELEASE_3_PAPERS_AND_STARTERS.zip`: three papers, three starter projects and the briefing).

> **How to use this.** The briefing says: *attempt each starter without consulting solutions; afterwards compare with the staff reference only where your lecturer authorises.* These are **my** model answers, not the staff reference. Do each paper on a 100-minute timer first, then read the matching Part. Every number the papers give was reproduced by running the logic of these solutions (Spring itself could not be downloaded where I built them, so run each project once in IntelliJ yourself).

---

## Contents

| Part | What |
| :-- | :-- |
| A | The exam's shape and a 100-minute plan |
| B | Java toolkit: every technique the papers use |
| C | SIMPLE paper: Parking (worked solution) |
| D | MODERATE paper: Dispatch (worked solution) |
| E | EXTREME paper: Scheduler (worked solution) |
| F | Section B testing and submission |
| G | Final checklist and one-page sheet |

---

## Part A: The exam's shape and a plan

## A1. What every paper has in common

| | All three practice papers |
| :-- | :-- |
| Time | 100 minutes, 100 marks |
| Section A (80) | Task 1: complete the first subclass (18). Task 2: create the second subclass + polymorphic processing in the service (22). Task 3: service methods, validation, errors (20). Task 4: controller endpoints (20). |
| Section B (20) | Six functional checks of the **running** app (3+3+3+3+4+4) |
| Stack | Java 17, Spring Boot 3.3.5, Maven, port 8081/8082/8083, no Swagger |
| Provided, do not replace | Abstract parent class, DTO records, repository API, constructor injection |
| You edit | The `// TODO`s, plus one new model class, plus the seed line(s) in the repository |
| Rules repeated in every paper | No `instanceof`/branching on concrete types; validate input; overflow-safe arithmetic; 404 + 400 via `ApiError`; POST returns 201; controllers delegate to the service |

Note: in the papers, Task 3 asks for the **handler** for 404 and 400 (in the EXTREME paper this moves to Task 4 together with 409).

## A2. The 100-minute plan

| Clock | Do |
| :-- | :-- |
| **0:00–0:05** | Unzip, open the folder with `pom.xml`, wait for Maven, run once (S1 §12). Note the port. |
| **0:05–0:15** | Read the whole paper. Read **every** starter file: parent class (abstract methods, constructor order, getters), DTO records (component names), repository (method names, what `get` returns), service TODOs, controller TODOs, exception classes (constructor parameters), `ApiError`. Write the illustrative numbers on paper: they are your test data. |
| **0:15–0:30** | **Task 1.** Restart; `GET` the list; call the subclass's method through a quick check. |
| **0:30–0:50** | **Task 2.** New class, seed line, service processing. Restart; check the illustrative calculation. |
| **0:50–1:05** | **Task 3** (service + handler). Restart; check 404 and 400. **Upload a safety ZIP.** |
| **1:05–1:25** | **Task 4** (controller). Restart; run every request in the paper's "REST operations" list. |
| **1:25–1:30** | Run all six Section B checks (Part F). Fix only what fails. |
| **1:30–1:40** | Stop coding. ZIP the whole project, open the ZIP to check it, upload, submit. |

**Milestones beat perfection** (briefing: *practise getting partial working milestones*). The app must **start** for Section B to score anything. If a method is unfinished, leave it compiling (keep the `throw new UnsupportedOperationException(...)`) rather than half-typed.

**Partial credit:** Section A is marked criterion by criterion. A Task 2 class with the right fields, constructor and getter but a wrong formula still scores the structure marks.

## A3. A fast trick for testing while you work

Before Task 4 exists, you cannot call the service through HTTP. You can still check a model class from the existing `GET` endpoint (it lists the objects), or temporarily add a `main` method to a scratch class:

```java
public class Scratch {
    public static void main(String[] args) {
        System.out.println(new HourlyPass(1, "CAR-101", 250).priceCents(61));   // expect 500
    }
}
```

Run it with its green arrow, then delete the class before zipping.

---

## Part B: Java toolkit

## B1. Records (DTOs)

Covered fully in S2 §9. The three things to remember:

```java
public record NewHourlyPass(long id, String registration, int rateCents) {}

body.id()                 // accessor: no "get"
new Quote(1, "CAR-101", "HourlyPass", 61, 500)      // canonical constructor, components in order
a.equals(b)               // true when every component is equal
```

## B2. The conditional (ternary) operator

`condition ? valueIfTrue : valueIfFalse` is a one-line `if/else` that **produces a value**:

```java
return medical ? 3 : 1;
```

Read: "if `medical`, then 3, otherwise 1". Exactly the same as:

```java
if (medical) {
    return 3;
}
return 1;
```

Use whichever you find clearer. The solutions use the `if` form.

## B3. Integer division rounding **up** (ceiling division)

`int / int` drops the decimal (Pre-Phase 0 §7.2): `61 / 60` is `1`. The parking paper needs **1–60 minutes = 1 hour, 61–120 = 2 hours**: rounding up.

```java
int hours = stayMinutes / 60;
if (stayMinutes % 60 != 0) {
    hours = hours + 1;          // any leftover minutes start another hour
}
```

| stayMinutes | `/ 60` | `% 60` | hours |
| :-- | :-- | :-- | :-- |
| 1 | 0 | 1 | 1 |
| 60 | 1 | 0 | 1 |
| 61 | 1 | 1 | 2 |
| 120 | 2 | 0 | 2 |

You may see the shortcut `(stayMinutes + 59) / 60`. It works for normal values but **overflows** when `stayMinutes` is near the `int` maximum (adding 59 wraps to a negative number), and the papers award marks for preventing overflow. `Math.ceilDiv` exists only from Java 18; your starters use Java 17.

## B4. Overflow: when numbers wrap around

An `int` holds −2,147,483,648 to **2,147,483,647** (`Integer.MAX_VALUE`). Go past the top and Java **silently wraps** to a negative number:

```java
int big = Integer.MAX_VALUE;
System.out.println(big + 1);          // -2147483648  (no error!)
```

The papers say *"prevent arithmetic overflow"*, *"long intermediate arithmetic and safe conversion to int"*, *"calculate energy before saving to catch overflow"*. Three tools:

| Tool | Meaning | On overflow |
| :-- | :-- | :-- |
| `Math.addExact(a, b)`, `Math.subtractExact(a, b)`, `Math.multiplyExact(a, b)` | Same as `+`, `-`, `*` | Throws `ArithmeticException` instead of wrapping |
| Do the maths in `long`: `2L * distance + 5L * load` | A `long` holds about ±9.2 × 10¹⁸; `int`-sized inputs cannot overflow it here | — |
| `Math.toIntExact(longValue)` | Convert `long` → `int` | Throws `ArithmeticException` if it does not fit |

| Piece | Meaning |
| :-- | :-- |
| `2L` | A `long` literal. `2L * x` makes the whole multiplication happen in `long`. (`2 * x` would be `int` maths, overflowing **before** any conversion.) |
| `(long) nowTick - getReleaseTick()` | A **cast**: turn `nowTick` into a `long` first, so the subtraction is done in `long`. |

**Then turn `ArithmeticException` into a 400**, because the handler only maps `IllegalArgumentException` (S4 §5.5):

```java
try {
    return Math.multiplyExact(hours, rateCents);
} catch (ArithmeticException e) {
    throw new IllegalArgumentException("Charge is too large to calculate");
}
```

## B5. Sorting with `Comparator`

A **`Comparator`** is an object that decides which of two items comes first. Streams use it in `.sorted(...)`.

### B5.1 Building one

```java
Comparator.comparingInt(Quote::priceCents)          // smaller price first
          .thenComparingLong(Quote::id)             // if prices tie, smaller id first
```

| Piece | Meaning |
| :-- | :-- |
| `Comparator.comparingInt(f)` | Compare items by the `int` that `f` returns, ascending (ASC = smallest first). |
| `Quote::priceCents` | A **method reference**: shorthand for `q -> q.priceCents()`. |
| `.thenComparingLong(f)` | A **tie-breaker**, used only when the previous comparison says "equal". |
| `comparingInt` / `comparingLong` / `comparing` | Use the one matching the value's type: `int`, `long`, or anything else (e.g. `String`). The `thenComparing...` versions follow the same rule. |

### B5.2 Descending (DESC = largest first) and the `reversed()` trap

```java
Comparator.comparingInt(DispatchMission::priority).reversed()     // priority DESC
          .thenComparingInt(DispatchMission::energyUnits)          // then energy ASC
          .thenComparingLong(DispatchMission::getId)               // then id ASC
```

`.reversed()` reverses **everything built so far**. Placed straight after the first key, it reverses only that key. Placed at the end, it would reverse all three:

```java
// WRONG for "priority DESC, energy ASC, id ASC":
Comparator.comparingInt(DispatchMission::priority)
          .thenComparingInt(DispatchMission::energyUnits)
          .thenComparingLong(DispatchMission::getId)
          .reversed();                     // now energy and id are DESC too
```

### B5.3 Sorting by a calculated value with a parameter

`urgencyAt(nowTick)` takes an argument, so it cannot be a method reference. Use a lambda, and tell Java the type:

```java
Comparator.comparingInt((Workload w) -> w.urgencyAt(nowTick)).reversed()
          .thenComparingLong(Workload::getId)
```

`(Workload w) -> ...` declares the lambda parameter's type. Without it, Java cannot always work out the type once `.reversed()` is chained, and you get a confusing compile error ("cannot find symbol" on `urgencyAt`).

### B5.4 Using it

```java
List<Quote> sorted = repo.all().stream()
        .map(pass -> toQuote(pass, minutes))
        .sorted(Comparator.comparingInt(Quote::priceCents).thenComparingLong(Quote::id))
        .toList();
```

`.toList()` (Java 16+) returns an **unmodifiable** list: fine to return as JSON.

## B6. Small JSON objects with `Map.of`

The dispatch paper wants *"a JSON object containing energyUnits"* but the service returns an `int`. Wrap it in the controller:

```java
@GetMapping("/{id}/energy")
public Map<String, Object> energy(@PathVariable long id) {
    return Map.of("id", id, "energyUnits", service.energy(id));
}
```

→ `{"id":101,"energyUnits":39}` (the key order may vary).

| Piece | Meaning |
| :-- | :-- |
| `Map.of(k1, v1, k2, v2, ...)` | An unmodifiable map from key/value pairs; up to 10 pairs. |
| `Map<String, Object>` | Keys are text; values can be anything. |
| No `null` keys or values | `Map.of` throws `NullPointerException` on `null`. |

Jackson turns a map into a JSON object whose fields are the keys.

## B7. Other helpers you will meet

| Code | Meaning |
| :-- | :-- |
| `Math.max(a, b)` / `Math.min(a, b)` | The larger / smaller of two numbers. `Math.max(0L, x)` = "x, but never below 0". |
| `Integer.MAX_VALUE` | The largest `int`: used as "no deadline". |
| `obj.getClass().getSimpleName()` | The real class's short name, e.g. `"AirMission"`: a type label without branching. |
| `text.trim()` | Remove spaces from both ends. |
| `text.isBlank()` | Empty or only spaces. Check `null` first. |
| `List.copyOf(list)` | An unmodifiable copy. |
| `10_000` | Underscores in number literals are allowed for readability: same as `10000`. |
| `synchronized` on a method | Only one request at a time may run any `synchronized` method of that object. Used when several steps must happen together without another request interfering. |

## B8. Greedy selection

A **greedy algorithm** goes through items in a fixed order and takes each one if it fits **right now**, never reconsidering. The dispatch paper:

```java
int remaining = capacity;
for (DispatchMission m : ordered) {
    int e = m.energyUnits();
    if (e <= remaining) {
        selected.add(...);           // take it
        remaining = remaining - e;
    } else {
        skipped.add(m.getId());      // skip it, remember the order
    }
}
```

It does not search for the best combination; it is fast and **deterministic** (same input, same output) because the order is fully defined by the sort.

## B9. Idempotency

An operation is **idempotent** if doing it twice has the same effect as doing it once. The EXTREME paper's commit must be: a client that retries (because a network reply was lost) must not book everything twice. The usual technique: the client sends a unique **key** (`runKey`); the server remembers what it did for that key and, on a repeat, returns the stored result instead of doing it again. Same key with **different** data is an error: 409 Conflict.

---

## Part C: SIMPLE paper: Parking (worked solution)

**Scenario:** an `HourlyPass` and a `WeekendPass`, each pricing a stay differently; a REST API for quotes and creating hourly passes. Money in integer **cents**.

**Starter facts to read first:**

| File | Key facts |
| :-- | :-- |
| `ParkingPass` | abstract; `private final long id`, `String registration`; `protected ParkingPass(long id, String registration)`; `getId()`, `getRegistration()`; `public abstract int priceCents(int stayMinutes)` |
| `HourlyPass` | field, constructor and getter already there; `priceCents` throws "Task 1" |
| DTOs | `ApiError(String code, String message)`, `NewHourlyPass(long id, String registration, int rateCents)`, `Quote(long id, String registration, String passType, int stayMinutes, int priceCents)` |
| `PassRepository` | `Map` with `all()`, `get(id)` (→ `null` if missing), `contains(id)`, `save(pass)`; seed comment for Task 2 |
| `PassNotFoundException(long id)` | message `"Parking pass not found: " + id` |
| `ParkingService` | `quote`, `allQuotes`, `addHourly` throw; "you may add a private helper for Quote" |
| Controller | base `/api/passes`, `GET` list exists |

### C1. Task 1: `HourlyPass.priceCents` [18]

Mark scheme: keep inheritance, final field, constructor chaining, getter (4); ceiling division (6); multiply by rate (4); reject ≤ 0 and prevent overflow (4).

```java
@Override
public int priceCents(int stayMinutes) {
    if (stayMinutes <= 0) {
        throw new IllegalArgumentException("stayMinutes must be greater than zero");
    }
    int hours = stayMinutes / 60;
    if (stayMinutes % 60 != 0) {
        hours = hours + 1;
    }
    try {
        return Math.multiplyExact(hours, rateCents);
    } catch (ArithmeticException e) {
        throw new IllegalArgumentException("Charge is too large to calculate");
    }
}
```

Check against the paper: 60 → 1 h × 250 = **250**; 61 → 2 h × 250 = **500**. Add `@Override` (the starter omits it; it costs nothing and guards the signature).

### C2. Task 2: `WeekendPass` + `allQuotes` [22]

**New file** `model/WeekendPass.java` (right-click `model` → New → Java Class):

```java
package edu.practice.parking.model;

public class WeekendPass extends ParkingPass {

    private static final int FLAT_MINUTES = 480;
    private static final int EXTRA_HOUR_CENTS = 150;

    private final int flatCents;

    public WeekendPass(long id, String registration, int flatCents) {
        super(id, registration);
        this.flatCents = flatCents;
    }

    public int getFlatCents() {
        return flatCents;
    }

    @Override
    public int priceCents(int stayMinutes) {
        if (stayMinutes <= 0) {
            throw new IllegalArgumentException("stayMinutes must be greater than zero");
        }
        if (stayMinutes <= FLAT_MINUTES) {
            return flatCents;
        }
        int extraMinutes = stayMinutes - FLAT_MINUTES;
        int extraHours = extraMinutes / 60;
        if (extraMinutes % 60 != 0) {
            extraHours = extraHours + 1;      // each commenced hour counts
        }
        try {
            return Math.addExact(flatCents, Math.multiplyExact(extraHours, EXTRA_HOUR_CENTS));
        } catch (ArithmeticException e) {
            throw new IllegalArgumentException("Charge is too large to calculate");
        }
    }
}
```

| Line | Why |
| :-- | :-- |
| `extends ParkingPass` | IS-A pass; goes in the same map and list. |
| `super(id, registration)` | Parent's fields are `private final`; only its constructor can set them. |
| `private final int flatCents` + getter | Immutable, like the starter's `HourlyPass`; the getter also puts `flatCents` in the JSON. |
| Constants | Named numbers (OOP Phase 2). |
| 480 → flat; 481 → 1 extra commenced hour | 800 + 150 = **950**. |

**Seed** (repository constructor, uncomment/add):

```java
passes.put(2L, new WeekendPass(2, "VAN-202", 800));
```

**Service** (polymorphic, sorted, no `instanceof`):

```java
public List<Quote> allQuotes(int minutes) {
    validateMinutes(minutes);
    return repo.all().stream()
            .map(pass -> toQuote(pass, minutes))
            .sorted(Comparator.comparingInt(Quote::priceCents)
                    .thenComparingLong(Quote::id))
            .toList();
}

private void validateMinutes(int minutes) {
    if (minutes <= 0) {
        throw new IllegalArgumentException("minutes must be greater than zero");
    }
}

private Quote toQuote(ParkingPass pass, int minutes) {
    return new Quote(
            pass.getId(),
            pass.getRegistration(),
            pass.getClass().getSimpleName(),
            minutes,
            pass.priceCents(minutes));          // polymorphic call
}
```

Imports needed in the service: `java.util.Comparator`, `java.util.List` (the starter's `import java.util.*;` already covers both).

Check: at 481 minutes, Weekend 2 = 950, Hourly 1 = 9 hours × 250 = **2,250** → Weekend first, as the paper states.

### C3. Task 3: `quote`, `addHourly`, handler [20]

```java
public Quote quote(long id, int minutes) {
    validateMinutes(minutes);
    ParkingPass pass = repo.get(id);
    if (pass == null) {
        throw new PassNotFoundException(id);
    }
    return toQuote(pass, minutes);
}
```

`addHourly`: the validation method shown in S4 §7 (five checks, then `trim`, then save).

Handler (`error/ApiExceptionHandler.java`): S4 §5.4, two methods, codes `"NOT_FOUND"` and `"BAD_REQUEST"`. The solution adds the optional third handler for malformed requests (S4 §7.1).

### C4. Task 4: endpoints [20]

```java
@GetMapping("/{id}/quote")
public Quote quote(@PathVariable long id, @RequestParam int minutes) {
    return service.quote(id, minutes);
}

@GetMapping("/quotes")
public List<Quote> quotes(@RequestParam int minutes) {
    return service.allQuotes(minutes);
}

@PostMapping("/hourly")
@ResponseStatus(HttpStatus.CREATED)
public ParkingPass addHourly(@RequestBody NewHourlyPass body) {
    return service.addHourly(body);
}
```

Every handler is one line that delegates: the 5 "correct annotations + delegation" marks.

### C5. Section B for Parking

| Check | Request | Expected |
| :-- | :-- | :-- |
| 1 | `GET /api/passes/1/quote?minutes=61` | `"priceCents":500` |
| 2 | `GET /api/passes/2/quote?minutes=481` | `"priceCents":950` |
| 3 | `GET /api/passes/quotes?minutes=61` | ids 1 (500) then 2 (800) |
| 4 | `GET /api/passes/99/quote?minutes=61` | 404, `"code":"NOT_FOUND"` |
| 5 | `minutes=0`; POST with blank registration or duplicate id | 400; list unchanged |
| 6 | `POST /api/passes/hourly {"id":3,"registration":"CAR-303","rateCents":300}` | 201; id 3 appears in later list/quotes |

All of these are in `section-b-checks.http` in the solution zip.

---

## Part D: MODERATE paper: Dispatch (worked solution)

**Scenario:** ground and air missions, each with its own energy formula and priority; plan a fixed energy budget greedily.

**Starter facts:** parent `DispatchMission` is abstract with `id`, `distanceKm`, `loadKg` (getters), and **two** abstract methods `int energyUnits()` and `int priority()`. Repository: `all()`, `find(id)` (→ `null`), `exists(id)`, `save(m)`. DTOs: `DispatchPlan(capacity, energyUsed, energyRemaining, List<PlanLine> selected, List<Long> skipped)`, `NewAirMission(id, distanceKm, loadKg, windPenalty, critical)`, `PlanLine(id, type, priority, energyUnits)`.

### D1. Task 1: `GroundMission` [18]

```java
@Override
public int energyUnits() {
    long energy = 2L * getDistanceKm() + 5L * getLoadKg();
    return Math.toIntExact(energy);
}

@Override
public int priority() {
    if (medical) {
        return 3;
    }
    return 1;
}
```

Check: 101 → 2×12 + 5×3 = **39**, medical → priority **3**.

### D2. Task 2: `AirMission` + `plan` [22]

**New file** `model/AirMission.java`:

```java
package edu.practice.dispatch.model;

public class AirMission extends DispatchMission {

    private final int windPenalty;
    private final boolean critical;

    public AirMission(long id, int distanceKm, int loadKg, int windPenalty, boolean critical) {
        super(id, distanceKm, loadKg);
        this.windPenalty = windPenalty;
        this.critical = critical;
    }

    public int getWindPenalty() { return windPenalty; }
    public boolean isCritical() { return critical; }

    @Override
    public int energyUnits() {
        long energy = 3L * getDistanceKm() + 8L * getLoadKg() + 2L * windPenalty;
        return Math.toIntExact(energy);
    }

    @Override
    public int priority() {
        if (critical) {
            return 5;
        }
        return 2;
    }
}
```

The constructor's parameter order **must** match the seed line the paper gives: `AirMission(103, 8, 1, 3, true)` = id, distance, load, wind, critical. Check: 3×8 + 8×1 + 2×3 = **38**, critical → **5**.

**Seed:** `map.put(103L, new AirMission(103, 8, 1, 3, true));`

**Service `plan`:**

```java
public DispatchPlan plan(int capacity, boolean urgentOnly) {
    if (capacity <= 0) {
        throw new IllegalArgumentException("capacity must be greater than zero");
    }

    List<DispatchMission> ordered = repo.all().stream()
            .filter(m -> !urgentOnly || m.priority() >= 3)
            .sorted(Comparator.comparingInt(DispatchMission::priority).reversed()
                    .thenComparingInt(DispatchMission::energyUnits)
                    .thenComparingLong(DispatchMission::getId))
            .toList();

    int remaining = capacity;
    List<PlanLine> selected = new ArrayList<>();
    List<Long> skipped = new ArrayList<>();

    for (DispatchMission mission : ordered) {
        int energy = mission.energyUnits();
        if (energy <= remaining) {
            selected.add(new PlanLine(mission.getId(), mission.getClass().getSimpleName(),
                    mission.priority(), energy));
            remaining = remaining - energy;
        } else {
            skipped.add(mission.getId());
        }
    }
    return new DispatchPlan(capacity, capacity - remaining, remaining, selected, skipped);
}
```

| Line | Meaning |
| :-- | :-- |
| `.filter(m -> !urgentOnly \|\| m.priority() >= 3)` | Keep a mission if we are **not** in urgent-only mode, **or** it is urgent. One line covers both cases. |
| The comparator | priority DESC, energy ASC, id ASC (B5.2). |
| The loop | Greedy selection (B8). `skipped` keeps the order in which missions were skipped. |

**Trace at capacity 60** (paper's example):

| Order | Mission | Priority | Energy | Remaining before | Fits? | Remaining after |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| 1 | 103 Air | 5 | 38 | 60 | yes | 22 |
| 2 | 101 Ground | 3 | 39 | 22 | **no → skip** | 22 |
| 3 | 102 Ground | 1 | 18 | 22 | yes | 4 |

Result: selected [103, 102], skipped [101], used 56, remaining 4. Exactly the paper's numbers.

With `urgentOnly=true`, 102 (priority 1) is filtered out first.

### D3. Task 3: `energy`, `addAir`, handler [20]

```java
public int energy(long id) {
    DispatchMission mission = repo.find(id);
    if (mission == null) {
        throw new MissionNotFoundException(id);
    }
    return mission.energyUnits();
}
```

`addAir`: checks in order: body present; id > 0; distance > 0; load ≥ 0; wind ≥ 0; each ≤ 10,000; id not already present; build the `AirMission`; **call `energyUnits()` once inside try/catch** (turning `ArithmeticException` into `IllegalArgumentException`); then save. Full code in the solution's `DispatchService.java`.

Capacity ≤ 0 is rejected in `plan` (above). Handler: same two methods as Parking, with `MissionNotFoundException`.

### D4. Task 4: endpoints [20]

```java
@GetMapping("/{id}/energy")
public Map<String, Object> energy(@PathVariable long id) {
    return Map.of("id", id, "energyUnits", service.energy(id));
}

@PostMapping("/air")
@ResponseStatus(HttpStatus.CREATED)
public DispatchMission addAir(@RequestBody NewAirMission request) {
    return service.addAir(request);
}

@GetMapping("/plan")
public DispatchPlan plan(@RequestParam int capacity,
                         @RequestParam(defaultValue = "false") boolean urgentOnly) {
    return service.plan(capacity, urgentOnly);
}
```

`urgentOnly defaults to false` is the `defaultValue` (S2 §7). The plan endpoint is 10 marks: its JSON must contain `capacity`, `energyUsed`, `energyRemaining`, `selected`, `skipped`: the `DispatchPlan` record's components give exactly those keys.

### D5. Section B for Dispatch

| Check | Request | Expected |
| :-- | :-- | :-- |
| 1 | `GET /api/missions/101/energy` (+ plan line) | 39; priority 3 |
| 2 | `GET /api/missions/103/energy` | 38; priority 5 |
| 3 | `GET /api/missions/plan?capacity=60&urgentOnly=false` | selected 103, 102; skipped [101] |
| 4 | `...&urgentOnly=true` | 102 absent |
| 5 | `capacity=0`; invalid POST; `GET /api/missions/999/energy` | 400, 400, 404 |
| 6 | Valid POST of 104 | 201; later plans include 104 |

---

## Part E: EXTREME paper: Scheduler (worked solution)

The briefing calls this a stretch that *"intentionally goes beyond routine expectations"*. Do Parts C and D first. The structure is the same; the algorithm is harder.

**Starter facts:** abstract `Workload` (id, cpuCores, memoryMb, durationTicks, releaseTick; abstract `urgencyAt(int nowTick)` and `latestStartTick()`). Repository: `all()`, `find(id)`. DTOs: `PlanRequest(nowTick, horizon, cpuCapacity, memoryCapacity)`, `CommitRequest(runKey, ...)` with `asPlanRequest()`, `Reservation(id, startTick, endTickExclusive, cpuCores, memoryMb)`, `Rejection(id, reason)`, `SchedulePlan(nowTick, horizon, selected, rejected)`. Exceptions: `WorkloadNotFoundException(long)`, `CommitConflictException(String runKey)`. Service methods already declared `synchronized`.

### E1. Task 1: `BatchWorkload` [18]

```java
@Override
public int urgencyAt(int nowTick) {
    if (nowTick < 0) {
        throw new IllegalArgumentException("nowTick cannot be negative");
    }
    long waited = Math.max(0L, (long) nowTick - getReleaseTick());
    long urgency = 100L + 2L * waited;
    try {
        return Math.toIntExact(urgency);
    } catch (ArithmeticException e) {
        throw new IllegalArgumentException("Urgency is too large");
    }
}

@Override
public int latestStartTick() {
    return Integer.MAX_VALUE;
}
```

"Aging": the longer a batch job has waited since release, the more urgent it becomes. Workload 1 (release 0): tick 0 → 100, tick 5 → 110.

### E2. Task 2: `DeadlineWorkload` + ranking [22]

New class with an extra `deadlineTick` (constructor order from the seed line: id, cpu, memory, duration, release, **deadline**):

```java
@Override
public int urgencyAt(int nowTick) {
    if (nowTick < 0) {
        throw new IllegalArgumentException("nowTick cannot be negative");
    }
    long slack = Math.max(0L, (long) deadlineTick - nowTick - getDurationTicks());
    long urgency = 1000L - 5L * slack;
    try {
        return Math.toIntExact(urgency);
    } catch (ArithmeticException e) {
        throw new IllegalArgumentException("Urgency is out of range");
    }
}

@Override
public int latestStartTick() {
    return Math.subtractExact(deadlineTick, getDurationTicks());
}
```

**Slack** = spare time before you must start. Less slack → more urgent. Workload 2 at tick 0: slack = 4 − 0 − 2 = 2 → 1000 − 10 = **990**; latest start = 4 − 2 = **2**.

Seeds: `DeadlineWorkload(2,4,6000,2,0,4)` and `DeadlineWorkload(4,3,6000,3,1,6)`.

Ranking (urgency DESC, id ASC):

```java
public List<Workload> ranked(int nowTick) {
    if (nowTick < 0) {
        throw new IllegalArgumentException("nowTick cannot be negative");
    }
    return repo.all().stream()
            .sorted(Comparator.comparingInt((Workload w) -> w.urgencyAt(nowTick)).reversed()
                    .thenComparingLong(Workload::getId))
            .toList();
}
```

At tick 0: 2 (990), 4 (985), 1 (100), 3 (100). 1 and 3 tie; id breaks it. `urgency(id, nowTick)` is a find-or-404 like D3.

### E3. Task 3: preview scheduling [20]

**Half-open interval** `[start, end)`: includes `start`, excludes `end`. A job with start 2 and duration 3 occupies ticks 2, 3, 4; `endTickExclusive` = 5. Two jobs `[0,2)` and `[2,5)` do **not** overlap.

The algorithm for each workload, in ranked order:

1. Skip it if it already has a **confirmed** reservation (my design choice, so a later commit never books the same workload twice).
2. If it needs more CPU or memory than the whole capacity → reject **EXCEEDS_CAPACITY**.
3. `earliest = max(nowTick, releaseTick)`; `windowLatest = nowTick + horizon − duration`. If `earliest > windowLatest`: reject **RELEASE_OUTSIDE_WINDOW** if it is released after `nowTick`, otherwise **NO_SLOT** (it simply cannot fit in the window).
4. If `latestStartTick() < earliest` → reject **DEADLINE**.
5. `latest = min(windowLatest, latestStartTick())`. Try every start `s` from `earliest` to `latest`; take the **first** where, on **every** tick of `[s, s + duration)`, existing confirmed reservations + already-selected ones + this job fit within both CPU and memory capacity.
6. None fits → reject **NO_SLOT**. Otherwise add a `Reservation` to `selected` **and** to this preview's working list of occupied slots.

Preview must not change the confirmed list: it works on a **copy** (`new ArrayList<>(confirmed)`).

The capacity check, in the service:

```java
private boolean fits(List<Reservation> occupied, int start, int end, Workload w, PlanRequest req) {
    for (int tick = start; tick < end; tick++) {
        int cpu = w.getCpuCores();
        int mem = w.getMemoryMb();
        for (Reservation r : occupied) {
            if (r.startTick() <= tick && tick < r.endTickExclusive()) {
                cpu += r.cpuCores();
                mem += r.memoryMb();
            }
        }
        if (cpu > req.cpuCapacity() || mem > req.memoryCapacity()) {
            return false;
        }
    }
    return true;
}
```

`r.startTick() <= tick && tick < r.endTickExclusive()` is the half-open test: "is reservation `r` running at this tick?" The paper's warning (*check both resources on every tick, not merely at the start tick*) is exactly the outer loop.

**Trace** for `{"nowTick":0,"horizon":6,"cpuCapacity":6,"memoryCapacity":10000}`:

| Rank | Job | cpu/mem/dur, release, latest start | Tries | Result |
| :-- | :-- | :-- | :-- | :-- |
| 1 | 2 | 4 / 6000 / 2, rel 0, latest 2 | s=0: ticks 0–1 use 4 cpu | **[0,2)** |
| 2 | 4 | 3 / 6000 / 3, rel 1, latest 3 | s=1: tick 1 → 4+3 = 7 cpu ✗; s=2: ticks 2–4 free | **[2,5)** |
| 3 | 1 | 2 / 4000 / 3, rel 0, no deadline | s=0: tick 0 → 6 cpu, 10000 mem ✓; tick 1 ✓; tick 2 → 5 cpu, 10000 mem ✓ | **[0,3)** |
| 4 | 3 | 2 / 2000 / 2, rel 1 | s=1: 8 cpu ✗; s=2: 7 cpu ✗; s=3: ticks 3–4 → 5 cpu, 8000 mem ✓ | **[3,5)** |

All four selected, none rejected. Validation of the `PlanRequest` ranges (nowTick 0–1,000,000; horizon 1–24; cpu 1–64; memory 1–1,000,000) comes first and throws `IllegalArgumentException` → 400.

### E4. Task 4: commit, conflicts, REST [20]

```java
public synchronized SchedulePlan commit(CommitRequest request) {
    if (request == null || request.runKey() == null || request.runKey().isBlank()) {
        throw new IllegalArgumentException("runKey is required");
    }
    String key = request.runKey();
    if (commitRequests.containsKey(key)) {
        if (commitRequests.get(key).equals(request)) {
            return commitPlans.get(key);          // identical retry: original plan, nothing new booked
        }
        throw new CommitConflictException(key);   // same key, changed values: 409
    }
    SchedulePlan plan = preview(request.asPlanRequest());
    confirmed.addAll(plan.selected());
    commitRequests.put(key, request);
    commitPlans.put(key, plan);
    return plan;
}
```

| Piece | Why |
| :-- | :-- |
| Two maps keyed by `runKey` | Remember what each key asked for and what it produced (B9). |
| `commitRequests.get(key).equals(request)` | Records compare every component: "same request?" in one call. |
| `synchronized` | Preview + add + remember happen as one unit; two simultaneous commits cannot interleave ("atomically"). |
| `reservations()` sorted by start tick, then id | "Deterministic ordering in GET /reservations". |

The handler adds a third method: `CommitConflictException` → `HttpStatus.CONFLICT`, `new ApiError("CONFLICT", ...)`.

Controller endpoints: `GET /{id}/urgency?nowTick=` (a `Map.of` with id, nowTick, urgency), `GET /ranked?nowTick=`, `POST /preview` (`@RequestBody PlanRequest`), `POST /commit` (`@RequestBody CommitRequest`), `GET /reservations`. Commit returns 200 in my solution (the paper fixes no success code; 201 would also be defensible for the first commit).

**Path check:** `/ranked` and `/reservations` are literal paths; `/{id}/urgency` has two segments. No clash.

---

## Part F: Section B testing and submission

### F1. Run every check, every time

Each solution zip contains `section-b-checks.http`. Copy the idea into your own attempt: one `###` request per Section B line, with the expected result in its title. Before zipping:

1. **Restart** the app.
2. Run the requests **top to bottom** (some depend on earlier POSTs).
3. For each, look at the **status code** and the **specific value** the check names.

Remember the repository is in memory: **restarting resets the data**. Check 6 ("added pass appears later") must be done within one run.

### F2. Reading a failure quickly

| You see | Look at |
| :-- | :-- |
| 500 with `UnsupportedOperationException: Task 3` | That TODO is still the starter line. |
| 500 with `ArithmeticException` | Overflow not converted to `IllegalArgumentException`. |
| 500 with your not-found exception | Handler missing / wrong type / missing annotation. |
| 404 for a whole endpoint | Path typo, or not restarted. |
| 400 you did not expect | Query parameter missing or misspelled (`minute=` vs `minutes=`), or JSON key names differ from the record's components. |
| Two summaries / prices identical | Override missing; `@Override` would have told you. |

### F3. ZIP and submit

The papers: *zip the complete Maven project folder (including `pom.xml`, sources, resources and any test files). Do not submit only `.java` files. Confirm the ZIP has the latest files and can be extracted.*

1. Stop the app. File → Save All.
2. In your file manager, go to the folder **containing** the project folder.
3. Compress the **project folder** (the one with `pom.xml` inside).
4. Open the ZIP: check `pom.xml`, `src/main/java/...` and one file you changed last.
5. Upload; follow the exam's file-name rule; submit.

---

## Part G: Final checklist

### G1. Before you leave each task

**Task 1 / Task 2 (models)**

- [ ] `extends` the provided parent; parent untouched
- [ ] Constructor parameters in the **seed line's order**; `super(...)` first
- [ ] New fields `private final`; public getters (`isX` for booleans)
- [ ] **Every** abstract method overridden with `@Override`, exact signature
- [ ] Parent data via getters (`getDistanceKm()`), not field names
- [ ] Invalid input → `IllegalArgumentException`; overflow handled (`long`, `...Exact`, convert to `IllegalArgumentException`)
- [ ] Seed line(s) added in the repository
- [ ] Service processing calls the abstract method through the parent type; **no `instanceof`**; sorting exactly as specified

**Task 3 (service + errors)**

- [ ] Missing id → the provided not-found exception (check the repository returns `null`)
- [ ] Every validation rule its own `if`, all **before** `save`
- [ ] `trim()` where asked; duplicate id checked with `contains`/`exists`
- [ ] Handler: one `@ExceptionHandler` per type; `ResponseEntity.status(...).body(new ApiError("CODE", ex.getMessage()))`

**Task 4 (controller)**

- [ ] Paths exactly as written in the paper, under the class's `@RequestMapping`
- [ ] `@PathVariable` for ids, `@RequestParam` for options (with `defaultValue` where the paper says "defaults to")
- [ ] `@RequestBody` with the provided DTO record
- [ ] POST: `@ResponseStatus(HttpStatus.CREATED)`
- [ ] One line per method; no try/catch; no business logic

### G2. One-page sheet

```java
// MODEL
public class Sub extends Parent {
    private final int extra;
    public Sub(long id, String name, int extra) { super(id, name); this.extra = extra; }
    public int getExtra() { return extra; }
    @Override public int calculate(int x) {
        if (x <= 0) throw new IllegalArgumentException("x must be positive");
        long result = 3L * x + 2L * extra;                       // long maths
        try { return Math.toIntExact(result); }                  // safe int
        catch (ArithmeticException e) { throw new IllegalArgumentException("too large"); }
    }
}
// ceiling division: int h = n / 60; if (n % 60 != 0) h++;

// SERVICE
public List<Line> process(int x) {
    return repo.all().stream()
        .map(p -> new Line(p.getId(), p.getClass().getSimpleName(), p.calculate(x)))  // polymorphic
        .sorted(Comparator.comparingInt(Line::value).reversed()     // DESC
                .thenComparingLong(Line::id))                        // ASC tie-break
        .toList();
}
public Thing get(long id) {
    Thing t = repo.get(id);
    if (t == null) throw new ThingNotFoundException(id);
    return t;
}
public Thing add(NewThing b) {
    if (b == null) throw new IllegalArgumentException("body required");
    if (b.id() <= 0) throw new IllegalArgumentException("id must be positive");
    if (b.name() == null || b.name().isBlank()) throw new IllegalArgumentException("name required");
    if (repo.contains(b.id())) throw new IllegalArgumentException("duplicate id");
    Thing t = new Sub(b.id(), b.name().trim(), b.extra());
    repo.save(t);
    return t;
}

// HANDLER
@RestControllerAdvice
public class ApiExceptionHandler {
    @ExceptionHandler(ThingNotFoundException.class)
    public ResponseEntity<ApiError> nf(ThingNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(new ApiError("NOT_FOUND", ex.getMessage()));
    }
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ApiError> bad(IllegalArgumentException ex) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(new ApiError("BAD_REQUEST", ex.getMessage()));
    }
}

// CONTROLLER
@GetMapping("/{id}/calc")
public Map<String, Object> calc(@PathVariable long id, @RequestParam int x) {
    return Map.of("id", id, "value", service.calc(id, x));
}
@GetMapping("/plan")
public Plan plan(@RequestParam int cap, @RequestParam(defaultValue = "false") boolean urgentOnly) {
    return service.plan(cap, urgentOnly);
}
@PostMapping("/sub") @ResponseStatus(HttpStatus.CREATED)
public Thing add(@RequestBody NewThing body) { return service.add(body); }
```

| Remember | |
| :-- | :-- |
| Read every starter file first | Constructor order comes from the seed line |
| Restart after every change | Data resets on restart |
| `reversed()` reverses everything before it | Put it straight after the DESC key |
| `ArithmeticException` is a 500 | Convert to `IllegalArgumentException` |
| Safety ZIP after Task 3 | Final ZIP of the whole folder, then open it to check |

Good luck.
