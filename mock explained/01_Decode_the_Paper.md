# 01: Decode the Paper (How to Understand the Questions)

You said the paper felt unclear. That is normal: exam papers are written like **contracts**, not like explanations. Every sentence is a rule that a marker ticks off. Your job is to turn each sentence into one line of code.

## Part 1: The 5-step reading routine (use it on ANY paper)

Spend the first 10 minutes of the exam doing exactly this, on paper:

1. **Underline the nouns.** These become classes and fields. (*mission, ground, air, windPenalty, critical, capacity, plan*)
2. **Circle the verbs / "Implement X".** These become methods. (*energyUnits, priority, plan, energy, addAir*)
3. **Copy every number the paper gives** into a "test data" box. Paper examples are free test cases (39, 38, [103,102], 125...).
4. **Highlight every "must / do not / reject / return".** These are rules, and each one is probably a mark.
5. **Map each bullet to one place in the code.** If a bullet has no home yet, you're not done.

Then open the starter and **read every file before typing** (parent class, repository, exceptions, models). 5 minutes now saves 20 later.

## Part 2: Phrase dictionary (exam English → code)

| The paper says | It really means | Code |
| :-- | :-- | :-- |
| "chaining to the parent constructor using `super(...)`" | First line of your constructor passes id, distance, load up to the parent | `super(id, distanceKm, loadKg);` |
| "Initialise windPenalty and critical in the subclass" | The parent doesn't know these two; set them yourself after `super` | `this.windPenalty = windPenalty;` |
| "appropriate getters" | One `getX()` per field. Boolean gets `isX()` | `getWindPenalty()`, `isCritical()` |
| "Correctly override the abstract parent method" | Same name, same return type, same parameters, plus `@Override` | `@Override public int priority()` |
| "Use `long` for intermediate arithmetic and convert safely to int" | Multiply with `L` suffix, then `Math.toIntExact(...)` | `long e = 2L*d + 5L*l; return Math.toIntExact(e);` |
| "do not allow silent integer overflow" | `int` wraps to negative without error. Use the line above so it throws instead | same |
| "Do not use concrete subclass type checks" | Forbidden: `instanceof`, `if (m instanceof AirMission)`, casting. Just call `m.priority()` | polymorphism |
| "algorithm works polymorphically" | Loop over `DispatchMission` objects; Java picks the right `energyUnits()` itself | `for (DispatchMission m : list)` |
| "deterministic" | Same input → always same output. So the sort must have a **tie-breaker** (ID) | 3-key comparator |
| "greedy" | Go through the list in order. Take what fits. Never go back, never look ahead | single `for` loop |
| "Traverse the **complete** sorted list" | No `break`. Check every mission | |
| "otherwise add its ID to skipped and continue" | Doesn't fit → `skipped.add(id)`, then the next mission may still fit | |
| "filtered out must appear in neither selected nor skipped" | Filter **before** the loop, not inside the else branch | build `eligible` list first |
| "priority descending, then energy ascending, then ID ascending" | Highest priority first; ties → cheapest first; ties → smallest ID first | see comparator |
| "Reject duplicate IDs" | Check `repo.exists(id)` and throw | `IllegalArgumentException` |
| "Do not save an invalid mission" | All checks and the energy calculation must come **before** `repo.save(...)` | order of statements |
| "Invalid values must raise IllegalArgumentException for GlobalExceptionHandler to handle" | The **service** throws. The **handler** makes the HTTP response. The service never touches HTTP | `throw new IllegalArgumentException("...")` |
| "Map X to HTTP 400, ErrorResponse with code BAD_REQUEST" | `@ExceptionHandler` method returning status 400 and `new ErrorResponse("BAD_REQUEST", ex.getMessage())` | |
| "descriptive message" | Don't write `""`. Use `ex.getMessage()` | |
| "Do not return HTTP 200 for an error" | Must be 4xx. A try/catch that returns a normal body = lost marks | |
| "Keep controller methods focused on HTTP and delegating to DispatchService" | Controller methods are 1-line: `return service.x(...)`. No loops, no `if`, no maths | |
| "urgentOnly must default to false when omitted" | `@RequestParam(defaultValue = "false")` | |
| "Return HTTP 201 Created" | `@ResponseStatus(HttpStatus.CREATED)` on the POST method | |
| "Do not replace the supplied parent class, model contracts or repository API" | Don't rename or delete existing methods/classes. Only fill TODOs and add AirMission | |
| "Attempt all tasks, even if some parts do not compile" | Marker gives partial credit per criterion. But for Section B, the app must run, so keep it compiling | |

## Part 3: Reading the data tables (the part people misread)

### 3.1 "(103, 8, 1, 3, true)": which number is which?

A tuple is just constructor arguments **in constructor order**. Look at the constructor signature to decode it:

```
GroundMission(long id, int distanceKm, int loadKg, boolean medical)
  (101,        12,          3,             true)    → id 101, 12 km, 3 kg, medical
  (102,        4,           2,             false)   → id 102, 4 km, 2 kg, not medical

AirMission(long id, int distanceKm, int loadKg, int windPenalty, boolean critical)
  (103,      8,           1,         3,              true)  → id 103, 8 km, 1 kg, wind 3, critical
```

If you design the `AirMission` constructor in a different order from the paper, the seed line `new AirMission(103,8,1,3,true)` will not compile or will silently swap values. **Use exactly this order.** The order also matches `NewAirMission` JSON fields.

### 3.2 The "Illustrative calculations" table is your test suite

| Paper says | Do this by hand first |
| :-- | :-- |
| Ground 101: `2×12 + 5×3 = 39`, priority 3 | 24 + 15 = 39 ✔ medical → 3 |
| Air 103: `3×8 + 8×1 + 2×3 = 38`, priority 5 | 24 + 8 + 6 = 38 ✔ critical → 5 |
| Ground 102: `2×4 + 5×2 = 18`, priority 1 | (not stated in the paper, but you need it) 8 + 10 = 18, not medical → 1 |
| Air 104: `3×6 + 8×1 + 2×2 = 30`, priority 2 | 18 + 8 + 4 = 30, not critical → 2 |

### 3.3 Trace the plan with capacity 60 (do this on paper)

Missions with (priority, energy): 101 → (3, 39) · 102 → (1, 18) · 103 → (5, 38)

**Sort** priority DESC, energy ASC, ID ASC: `103 (5)`, `101 (3)`, `102 (1)` → `[103, 101, 102]`

| Step | Mission | Energy | Remaining before | Fits? | Action | Remaining after |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| 1 | 103 | 38 | 60 | 38 ≤ 60 yes | select | 22 |
| 2 | 101 | 39 | 22 | 39 ≤ 22 **no** | skip, **continue** | 22 |
| 3 | 102 | 18 | 22 | 18 ≤ 22 yes | select | 4 |

Result: selected `[103, 102]`, skipped `[101]`, used `56`, remaining `4`. This is Section B3.

**The trap:** a beginner reads "101 doesn't fit" and stops. The paper's whole point is that **102 is still picked after 101 is skipped**.

**With `urgentOnly=true`:** keep only priority ≥ 3 → 103 and 101 (102 has priority 1, it vanishes). Trace: 103 selected (rem 22), 101 skipped. Result: selected `[103]`, skipped `[101]`. 102 is in **neither**. (B4)

**After adding 104 (priority 2, energy 30), capacity 125:**
Sorted: 103 (5), 101 (3), 104 (2), 102 (1). Energies 38 + 39 + 30 + 18 = 125 → everything fits, order `[103, 101, 104, 102]`, used 125, remaining 0. (B6)

## Part 4: Task-by-task decoding

For each task: **what it says → what it means → what to write → where marks hide**.

### Task 1: GroundMission (18 marks)

**Says:** "Complete the `energyUnits()` TODO... `2 × distanceKm + 5 × loadKg`... use long... convert safely."
**Means:** Parent class stores distance and load as private fields with getters. You can't touch `distanceKm` directly; call `getDistanceKm()` and `getLoadKg()`. Do the maths in `long`, convert back.
```java
@Override
public int energyUnits() {
    long energy = (2L * getDistanceKm()) + (5L * getLoadKg());
    return Math.toIntExact(energy);
}
```
**Says:** "medical = 3, non-medical = 1."
```java
@Override
public int priority() {
    return medical ? 3 : 1;      // or if (medical) { return 3; } return 1;
}
```
**Marks hide in:** 12 marks for 1.1 is a lot for one formula, so expect them split: correct formula, `long` arithmetic, safe conversion, `@Override`. Missing the `L` or `toIntExact` loses part of it.
**Ignore:** "constructor and isMedical() already implemented." Don't touch them.

### Task 2.1: Create AirMission (7 marks)

**Says:** "Create `AirMission` in the model package, extending DispatchMission. Include fields, constructor, getters. Chain with super(...)."
**Means:** New file `model/AirMission.java`. The parent constructor is `protected DispatchMission(long id, int distanceKm, int loadKg)`, so your constructor must call it **first**.
```java
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
    // 2.2 methods go here
}
```
**Marks hide in:** `extends`, `super(...)` as the first line, the two private fields, the two getters. Structure marks are awarded even if the formula is wrong. Do 2.1 fast so everything else compiles.

### Task 2.2: AirMission calculations (7 marks)

Same pattern as Ground, with three terms. Priority: `critical ? 5 : 2`.
```java
long energy = (3L * getDistanceKm()) + (8L * getLoadKg()) + (2L * windPenalty);
return Math.toIntExact(energy);
```

### Task 2.3: `DispatchService.plan(capacity, urgentOnly)` (8 marks)

This is the "brain" task. Break it into **five small jobs**:

| Job | Code idea |
| :-- | :-- |
| 0. Reject bad capacity (this is Task 3.3's first bullet, but it lives here) | `if (capacity <= 0) throw new IllegalArgumentException(...)` |
| 1. Filter | Loop over `repo.all()`. Add to `eligible` if `!urgentOnly \|\| m.priority() >= 3` |
| 2. Sort | Priority DESC, energy ASC, ID ASC |
| 3. Greedy loop | For each: `energy <= capacity - used` → select; else `skipped.add(id)` |
| 4. Totals | `remaining = capacity - used` |
| 5. Return | `new DispatchPlan(capacity, used, remaining, selected, skipped)` |

**`PlanLine` fields:** `id`, `type`, `priority`, `energyUnits`. For `type` use `m.getClass().getSimpleName()` which gives `"GroundMission"` or `"AirMission"` without any `instanceof`.

**Marks hide in:** the filter position (before loop), all three sort keys, no `break`, no `instanceof`.

### Task 3.1: `energy(id)` (6 marks)

"Look it up; if absent throw `MissionNotFoundException`; else return its energy."
```java
DispatchMission m = repo.find(id);        // returns null when missing
if (m == null) throw new MissionNotFoundException(id);
return m.energyUnits();
```
Note: `repo.find` returns **`null`**, not `Optional`. Check the starter.

### Task 3.2: `addAir(NewAirMission request)` (7 marks)

**The key distinction:** `NewAirMission` = the **JSON request** (just data, no behaviour). `AirMission` = the **domain object** (has formulas). The service converts one into the other.

**Order of operations (this is what "do not save an invalid mission" means):**
1. Validate each range → throw `IllegalArgumentException` if bad
2. Reject duplicate ID
3. Build `new AirMission(...)` from the request
4. Call `energyUnits()` once (overflow check)
5. **Only now** `repo.save(mission)`
6. Return the mission

Ranges: id 1-10000 · distanceKm 1-10000 · loadKg 0-10000 · windPenalty 0-10000.
Watch the **lower bounds**: distance starts at **1** but load and wind start at **0**. This is the #1 copy-paste error.

**Careful:** `NewAirMission` has `getCritical()` (not `isCritical()`). Look at the class.

### Task 3.3: Capacity validation and GlobalExceptionHandler (7 marks)

Two parts: (a) capacity ≤ 0 → `IllegalArgumentException` (inside `plan`), (b) fill in the handler:

```java
@ExceptionHandler(IllegalArgumentException.class)
public ResponseEntity<ErrorResponse> handleBadRequest(IllegalArgumentException ex) {
    return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse("BAD_REQUEST", ex.getMessage()));
}

@ExceptionHandler(MissionNotFoundException.class)
public ResponseEntity<ErrorResponse> handleNotFound(MissionNotFoundException ex) {
    return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("NOT_FOUND", ex.getMessage()));
}
```
The code strings must be **exactly** `BAD_REQUEST` and `NOT_FOUND` (capitals, underscore). The starter class is already annotated `@RestControllerAdvice`; keep it.

### Task 4: Controller (20 marks)

**Says:** "Controller methods focused on handling HTTP and delegating."
**Means:** Each method is one line calling the service. Three endpoints:

| Endpoint | Annotation | Parameter | Return |
| :-- | :-- | :-- | :-- |
| `GET /missions/{id}/energy` | `@GetMapping("/{id}/energy")` | `@PathVariable("id") long id` | `Map.of("energyUnits", service.energy(id))` → `{"energyUnits":39}` |
| `POST /missions/air` | `@PostMapping("/air")` + `@ResponseStatus(HttpStatus.CREATED)` | `@RequestBody NewAirMission request` | the created mission |
| `GET /missions/plan` | `@GetMapping("/plan")` | `@RequestParam("capacity") int capacity`, `@RequestParam(name="urgentOnly", defaultValue="false") boolean urgentOnly` | `DispatchPlan` |

The class is already `@RequestMapping("/missions")`, so the paths in the annotations are only the **rest** (`"/air"`, not `"/missions/air"`).

**"Return a JSON object containing energyUnits":** the service returns an `int`, but JSON must be `{"energyUnits":39}`, an object. Wrap it with `Map.of`. (You can't use a record here, and a map avoids creating a new class.)

### Repository seed line

"Add AirMission(103,8,1,3,true) in the existing repository seed section. Do not add duplicate GroundMission objects."
One new line inside the `MissionRepository()` constructor: `map.put(103L, new AirMission(103, 8, 1, 3, true));`
(Key is `103L`: the map is `Map<Long, ...>`.) It only compiles **after** AirMission exists, which is why Task 2.1 comes first.

## Part 5: Section B decoded

Section B = the marker starts your app and sends six requests. **Independent of your source code**, so a correct response earns marks even if the code was messy.

| Ref | Request to send | Must see | Marks |
| :-- | :-- | :-- | :-- |
| B1 | `GET /missions/101/energy` | `{"energyUnits":39}`; and priority 3 (visible as `"priority":3` in a plan, e.g. `capacity=200`) | 3 |
| B2 | `GET /missions/103/energy` | `{"energyUnits":38}` | 3 |
| B3 | `GET /missions/plan?capacity=60&urgentOnly=false` | selected 103, 102; skipped [101] | 3 |
| B4 | `GET /missions/plan?capacity=60&urgentOnly=true` | selected 103; skipped [101]; 102 nowhere | 3 |
| B5 | `capacity=0` or `-5`; POST duplicate ID / out-of-range; GET `/missions/999/energy` | 400 + `BAD_REQUEST`; 404 + `NOT_FOUND` | 4 |
| B6 | POST mission 104 → 201; GET `/missions` contains 104; plan capacity 125 | `[103,101,104,102]`, used 125, remaining 0 | 4 |

All six are in `section-b-checks.http`. Note: B6 requires 104 to be **added in the same running session**; the repository is in-memory, so restarting the app forgets 104.

## Part 6: The traps (read twice)

| Trap | What happens | Fix |
| :-- | :-- | :-- |
| Using `break` when something doesn't fit | 102 never selected → B3 fails | `else { skipped.add(id); }` and keep looping |
| Filtering inside the loop's `else` | filtered missions land in `skipped` → B4 fails | build `eligible` first |
| `.reversed()` at the **end** of the comparator chain | energy and ID also flip to descending → wrong order | `comparingInt(priority).reversed()` then `.thenComparing...` |
| `compare(first.priority(), second.priority())` for priority | ascending, not descending | swap the arguments for DESC |
| `2 * getDistanceKm()` without `L` | int maths; overflow before conversion | `2L *` |
| Validation after `repo.save` | invalid mission stored | validate → build → calculate → save |
| load/wind lower bound 1 instead of 0 | a legal 0 is rejected | distance 1-10000; load, wind 0-10000 |
| Calling `new ErrorResponse("BadRequest", ...)` | code mismatch | `"BAD_REQUEST"`, `"NOT_FOUND"` |
| Returning `ResponseEntity.ok(error)` | HTTP 200 for an error | `.status(HttpStatus.BAD_REQUEST)` |
| Putting `if`/loops in the controller | violates "delegate to service" | keep one-liners |
| Using `instanceof` in `plan` | explicit rule violation | call `priority()` / `energyUnits()` |
| Records for request/response | explicitly banned | use the supplied classes |
| Creating a `dto` package | explicitly banned | none needed |
| Forgetting the `AirMission` seed line | B2, B3, B4, B6 all fail | add it right after creating the class |
| JSON `critical` and getter name | `NewAirMission` uses `getCritical()` | call `request.getCritical()` |
| Zip only the `.java` files | marker can't build | zip the **whole folder** with `pom.xml` |

## Part 7: "How do I know I understood the question?" test

After reading any task, finish this sentence out loud. If you can't, re-read:

> "The method `____` receives `____`, must return `____`, and must throw `____` when `____`."

Example for 3.1: "`energy` receives a `long id`, must return an `int` (the mission's energyUnits), and must throw `MissionNotFoundException` when no mission has that ID."
