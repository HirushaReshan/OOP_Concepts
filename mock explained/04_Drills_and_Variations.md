# 04: Drills and Variations

Do the drills on paper, then check the answers. The point is **speed and certainty** on the three things the marker tests: formulas, the greedy trace, and error codes.

> The paper itself says: *"Sample tasks do not predict the real assessment."* The live exam will probably have a **different story with the same skeleton**. Part C shows you how to re-skin.

---

## Part A: Hand-trace drills

### A1: Formulas (1 min each)

Using the mock's formulas:
1. `GroundMission(id 7, 10 km, 4 kg, medical = false)`: energy? priority?
2. `AirMission(id 8, 5 km, 3 kg, wind 10, critical = true)`: energy? priority?
3. `AirMission(id 9, 1 km, 0 kg, wind 0, critical = false)`: energy? priority?
4. Largest legal Air energy (all inputs at 10,000)?

<details><summary>Answers</summary>

1. 2·10 + 5·4 = **40**, priority **1**
2. 3·5 + 8·3 + 2·10 = 15 + 24 + 20 = **59**, priority **5**
3. 3 + 0 + 0 = **3**, priority **2**
4. 30,000 + 80,000 + 20,000 = **130,000** (fits in `int`: why the paper says "valid values fit within int")
</details>

### A2: The greedy trace

Missions as `(id, priority, energy)`: **(1, 3, 30)**, **(2, 5, 50)**, **(3, 1, 10)**, **(4, 2, 20)**, **(5, 3, 25)**

Fill in for each: sorted order, selected IDs, skipped IDs, used, remaining.

a. capacity **80**, urgentOnly = false
b. capacity **80**, urgentOnly = true
c. capacity **200**, urgentOnly = false
d. capacity **45**, urgentOnly = false

<details><summary>Answers</summary>

Sorted (all five): priority DESC, then energy ASC → **[2, 5, 1, 4, 3]** (2 is p5; then p3: id 5 (25) before id 1 (30); then p2: id 4; then p1: id 3).

a. 2 (50) ✔ rem 30 → 5 (25) ✔ rem 5 → 1 (30) ✗ → 4 (20) ✗ → 3 (10) ✗. **selected [2,5], skipped [1,4,3], used 75, remaining 5.**
b. Only p ≥ 3: 2, 5, 1 → 2 ✔, 5 ✔, 1 ✗. **selected [2,5], skipped [1]**, used 75, remaining 5. IDs 4 and 3 appear in **neither** list.
c. Everything fits: 50 + 25 + 30 + 20 + 10 = 135. **selected [2,5,1,4,3], skipped [], used 135, remaining 65.**
d. 2 (50 > 45) ✗ skip → 5 (25) ✔ rem 20 → 1 (30) ✗ → 4 (20 ≤ 20) ✔ rem 0 → 3 (10 > 0) ✗. **selected [5,4], skipped [2,1,3], used 45, remaining 0.** (Shows two things: the *first* mission can be skipped, and "exactly fits" uses `<=`.)
</details>

### A3: Tie-breaks

Two missions, both priority 3 and energy 30, with IDs **9** and **4**. Which comes first?

<details><summary>Answer</summary>
ID **4**: all else equal, ID ascending. Sorted: [4, 9]. Without the ID tie-breaker the order would depend on the map's internal order. That is what "deterministic" prevents.
</details>

### A4: The mock's own data, new capacities

Mock missions after adding 104: `101 (p3, 39)`, `102 (p1, 18)`, `103 (p5, 38)`, `104 (p2, 30)`.

a. capacity **50**, urgentOnly=false
b. capacity **70**, urgentOnly=false
c. capacity **125**, urgentOnly=true
d. capacity **38**, urgentOnly=true

<details><summary>Answers</summary>

Sorted: [103, 101, 104, 102].
a. 103 ✔ (rem 12) → 101 ✗ → 104 ✗ → 102 ✗. **selected [103], skipped [101,104,102], used 38, rem 12.**
b. 103 ✔ (rem 32) → 101 (39) ✗ → 104 (30 ≤ 32) ✔ (rem 2) → 102 ✗. **selected [103,104], skipped [101,102], used 68, rem 2.**
c. Eligible: 103, 101. Both fit (77). **selected [103,101], skipped [], used 77, rem 48.** (104 and 102 in neither list.)
d. Eligible: 103, 101. 103 (38 ≤ 38) ✔ rem 0 → 101 ✗. **selected [103], skipped [101], used 38, rem 0.**
</details>

### A5: Predict the HTTP response

| # | Request | Status | Body (key part) |
| :-- | :-- | :-- | :-- |
| 1 | `GET /missions/102/energy` | ? | ? |
| 2 | `GET /missions/5000/energy` | ? | ? |
| 3 | `GET /missions/plan?capacity=-1` | ? | ? |
| 4 | `POST /missions/air` with `{"id":10001,...}` | ? | ? |
| 5 | `POST /missions/air` with valid new id 200 | ? | ? |
| 6 | `GET /missions/plan?capacity=60` (no urgentOnly) | ? | ? |

<details><summary>Answers</summary>

1. **200**, `{"energyUnits":18}`
2. **404**, `{"code":"NOT_FOUND","message":"Mission not found: 5000"}`
3. **400**, `{"code":"BAD_REQUEST","message":"Capacity must be greater than zero."}`
4. **400**, `{"code":"BAD_REQUEST","message":"id must be between 1 and 10000."}` (message wording is yours, the code string isn't)
5. **201**, the created mission as JSON (id, distanceKm, loadKg, windPenalty, critical)
6. **200**, same as `urgentOnly=false` (default)
</details>

---

## Part B: Find the bug

Each snippet has **one** mistake from the traps list. Find it before opening the answer.

**B1**
```java
for (DispatchMission m : eligible) {
    int e = m.energyUnits();
    if (e <= capacity - used) { selected.add(line(m)); used += e; }
    else { skipped.add(m.getId()); break; }
}
```
**B2**
```java
eligible.sort(Comparator.comparingInt(DispatchMission::priority)
        .thenComparingInt(DispatchMission::energyUnits)
        .thenComparingLong(DispatchMission::getId)
        .reversed());
```
**B3**
```java
for (DispatchMission m : repo.all()) {
    if (m.priority() >= 3 || !urgentOnly) { eligible.add(m); }
}
...
else { skipped.add(m.getId()); }   // what if urgentOnly filtered inside this else instead?
```
(Variant: the filter written as `if (urgentOnly && m.priority() < 3) { skipped.add(m.getId()); continue; }`)

**B4**
```java
public AirMission(long id, int distanceKm, int loadKg, int windPenalty, boolean critical) {
    this.windPenalty = windPenalty;
    this.critical = critical;
    super(id, distanceKm, loadKg);
}
```
**B5**
```java
public int energyUnits() {
    return 3 * getDistanceKm() + 8 * getLoadKg() + 2 * windPenalty;
}
```
**B6**
```java
AirMission mission = new AirMission(...);
repo.save(mission);
mission.energyUnits();
```
**B7**
```java
@ExceptionHandler(IllegalArgumentException.class)
public ResponseEntity<ErrorResponse> bad(IllegalArgumentException ex) {
    return ResponseEntity.ok(new ErrorResponse("BAD_REQUEST", ex.getMessage()));
}
```
**B8**
```java
@GetMapping("/missions/plan")
public DispatchPlan plan(@RequestParam int capacity, @RequestParam boolean urgentOnly) { ... }
```
(Class already has `@RequestMapping("/missions")`.)

**B9**
```java
if (m instanceof AirMission) { e = ((AirMission) m).energyUnits(); } else { e = m.energyUnits(); }
```
**B10**
```java
if (request.getLoadKg() < 1 || request.getLoadKg() > 10000) { throw new IllegalArgumentException("..."); }
```

<details><summary>Answers</summary>

- **B1:** `break` stops the greedy pass at the first misfit; later cheaper missions never get selected (B3 would select only [103]). Remove `break`.
- **B2:** `.reversed()` at the end flips all three keys. Put it right after `comparingInt(priority)`.
- **B3:** Putting urgent-filtered missions in `skipped` violates "filtered out appear in neither". The filter must only decide membership of `eligible`; skipped is only for eligible missions that didn't fit. (The first snippet `priority >= 3 || !urgentOnly` is correct, same logic reordered.)
- **B4:** `super(...)` must be the first statement; compile error.
- **B5:** Plain `int` arithmetic: no `L`, no `toIntExact`. Overflows silently; loses the safety marks.
- **B6:** Saved **before** calculating. "Calculate the energy before saving. Do not save an invalid mission."
- **B7:** `ResponseEntity.ok(...)` = HTTP 200. Use `.status(HttpStatus.BAD_REQUEST).body(...)`.
- **B8:** Path becomes `/missions/missions/plan`. Also `urgentOnly` is required with no default, so omitting it fails. Use `@GetMapping("/plan")` and `defaultValue = "false"`.
- **B9:** `instanceof` and cast: forbidden. Just call `m.energyUnits()`.
- **B10:** loadKg's minimum is **0**, not 1 (distance is the one that starts at 1).
</details>

---

## Part C: If the real exam is a different story

The skeleton is probably fixed (your old S6 file describes three earlier practice papers that shared it: Parking, Dispatch, Scheduler; names in the table below come from there, so treat them as illustrations). Learn the **skeleton**, not the story.

### The skeleton

| Slot | Mock (Dispatch) | Parking-style | Scheduler-style |
| :-- | :-- | :-- | :-- |
| Abstract parent | `DispatchMission` | `ParkingPass` | `Workload` |
| Subclass 1 (given, methods to finish) | `GroundMission` | `HourlyPass` | ... |
| Subclass 2 (you create) | `AirMission` | `WeekendPass` | ... |
| Abstract method 1 (a calculation) | `energyUnits()` | `priceCents(minutes)` | `urgencyAt(tick)` |
| Abstract method 2 (a rank/flag) | `priority()` | - | - |
| Service "brain" | greedy `plan` | sorted `quote` list | greedy scheduler |
| Lookup → 404 | `energy(id)` | `quote(id)` | `get(id)` |
| Add → validate → 201 | `addAir` | `addHourly` | `addX` |
| Handler | 400 + 404 | 400 + 404 | 400 + 404 (+ maybe 409) |

### Variations to be ready for (and the one-line adaptation)

| Variation | What changes |
| :-- | :-- |
| Different formula | Change the arithmetic only. Keep `long` + `toIntExact` |
| Different filter (e.g. `priority >= 4`, `type == medical`) | Change the `if` in the filter loop; **still no `instanceof`**, so use an abstract method or a getter on the parent |
| Different sort keys or directions | One `compare` step per key; ASC = `(first, second)`, DESC = swapped. Always end with ID ascending |
| "Stop at the first mission that doesn't fit" | Opposite of this paper: `break`. **Read the sentence carefully; this paper says continue** |
| Ceiling division (e.g. billing by started hour) | `h = minutes / 60; if (minutes % 60 != 0) h++;` (avoid `(m + 59) / 60`, it overflows near `Integer.MAX_VALUE`) |
| Duplicate ID → **409 Conflict** instead of 400 | Needs a `SomethingConflictException` + `@ExceptionHandler` returning `HttpStatus.CONFLICT` and code `"CONFLICT"` (create the exception in the `exception` package; extend `RuntimeException`; call `super(message)`) |
| Time-based calculation with a parameter (`nowTick`) | `@RequestParam` for it; pass to the method; cast to `long` early: `(long) now - getReleaseTick()` |
| Return type with extra fields | Add a getter + field to the supplied response class; don't create a `dto` package |
| `Map.of` for tiny JSON objects | `Map.of("energyUnits", n)`. For multiple keys: `Map.of("id", id, "energyUnits", n)` |
| Records instead of classes (other papers) | Check the paper's instructions. If records: accessor `x()`, not `getX()` |

### The universal build order (works on any paper of this family)

1. Read paper + **all** starter files. Note constructor parameter order and getter names.
2. Finish the given subclass's TODOs.
3. Create the new subclass; add the seed in the repository.
4. Service: lookup (404), add (validate → exists → build → calculate → save), main algorithm.
5. Exception handler(s).
6. Controller one-liners.
7. Run every request from the paper's "REST operations" and Section B tables.
8. Zip the whole project and verify the ZIP.

---

## Part D: Short-answer / viva questions (in case it is asked)

<details><summary>1. Why is <code>DispatchMission</code> abstract?</summary>
Because a generic "mission" has no energy formula or priority rule of its own; only concrete types do. Abstract methods force every subclass to supply them, and prevent creating a meaningless plain mission.
</details>

<details><summary>2. How does polymorphism help in <code>plan</code>?</summary>
The method works on <code>DispatchMission</code> references and calls <code>energyUnits()</code>/<code>priority()</code>; the JVM runs the right subclass version at runtime. A new mission type needs no change to <code>plan</code>.
</details>

<details><summary>3. <code>NewAirMission</code> vs <code>AirMission</code>?</summary>
<code>NewAirMission</code> is a request model: plain data Jackson fills from JSON. <code>AirMission</code> is the domain object with behaviour (the formulas), stored in the repository. The service validates the first and builds the second.
</details>

<details><summary>4. Why do validation and the greedy algorithm live in the service, not the controller?</summary>
Single responsibility / layering: the controller only handles HTTP, the service owns business rules, the repository only stores. It keeps rules testable without HTTP and lets the exception handler centralise HTTP error mapping.
</details>

<details><summary>5. Why <code>long</code> and <code>Math.toIntExact</code>?</summary>
<code>int</code> arithmetic wraps around silently on overflow. Doing maths in <code>long</code> avoids overflow during calculation, and <code>toIntExact</code> throws instead of silently truncating when narrowing back to <code>int</code>.
</details>

<details><summary>6. How does an <code>IllegalArgumentException</code> thrown in the service become an HTTP 400?</summary>
Nothing catches it in the service/controller, so it propagates up to Spring MVC, which finds the matching <code>@ExceptionHandler</code> in the <code>@RestControllerAdvice</code> class. That method returns a <code>ResponseEntity</code> with status 400 and an <code>ErrorResponse</code> body, which Jackson writes as JSON.
</details>

<details><summary>7. Why are results deterministic even though the repository uses a <code>ConcurrentHashMap</code>?</summary>
Map iteration order is not guaranteed, so the service copies the missions into a list and sorts them by a total order (priority DESC, energy ASC, ID ASC). IDs are unique, so no two missions ever tie fully.
</details>

<details><summary>8. What is dependency injection in this project?</summary>
The controller declares a <code>DispatchService</code> constructor parameter and the service declares a <code>MissionRepository</code>; Spring creates the beans and passes them in. We never call <code>new</code> on them, so everyone shares the same repository instance.
</details>

---

## Part E: Final self-audit (5 minutes before you stop)

Tick every line **by reading your code**, not from memory:

- [ ] `GroundMission.energyUnits`: `2L`, `5L`, `Math.toIntExact`, `@Override`
- [ ] `GroundMission.priority`: medical 3 / else 1, `@Override`
- [ ] `AirMission`: `extends DispatchMission`, `super(id, distanceKm, loadKg)` first, two fields, constructor, two getters
- [ ] `AirMission.energyUnits`: `3L`, `8L`, `2L`, `toIntExact`; `priority`: 5 / 2
- [ ] Repository seeds `103L → AirMission(103,8,1,3,true)`, no duplicate Ground seeds
- [ ] `plan`: capacity ≤ 0 throws; filter before sort; sort 3 keys; no `break`; no `instanceof`; skipped only for eligible-but-didn't-fit
- [ ] `PlanLine(id, type, priority, energyUnits)` filled correctly; `DispatchPlan` has capacity, used, remaining, selected, skipped
- [ ] `energy(id)`: null → `MissionNotFoundException`
- [ ] `addAir`: 4 ranges (1-10000, 1-10000, 0-10000, 0-10000), duplicate check, calculate, then save
- [ ] Handler: 400 + `"BAD_REQUEST"`, 404 + `"NOT_FOUND"`, messages from `ex.getMessage()`
- [ ] Controller: 3 endpoints, one-liners, 201 on POST, `defaultValue = "false"`, `GET /missions` kept
- [ ] No records, no `dto` package, main class still in base package
- [ ] App starts; all Section B requests give the expected output
- [ ] ZIP contains `pom.xml` + `src/` and I opened it to check
