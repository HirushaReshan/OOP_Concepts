# 03: Build It Step by Step (Exam Order)

Do this with the **starter zip** open in IntelliJ. Type the code yourself. Each step ends with a **checkpoint**. All code here was compiled and run against the paper's numbers (39, 38, `[103,102]`, `[103,101,104,102]`, 125/0).

**Package base:** `edu.practice.dispatch` → `controller`, `exception`, `model`, `repository`, `service`.

## Step 0: Open, run, look (5 min)

1. Unzip the starter. Open the folder that **contains `pom.xml`** in IntelliJ. Wait for Maven to finish.
2. Project JDK = **25**.
3. **Read these starter files before typing anything:** `DispatchMission` (abstract; note constructor + getters), `GroundMission`, `MissionRepository` (note `find` returns `null`), `MissionNotFoundException` (constructor takes a `long`), `NewAirMission` (note `getCritical()`), `DispatchPlan`, `PlanLine`, `ErrorResponse`.
4. The app **will not run usefully yet**: the TODO methods throw `UnsupportedOperationException`. That's fine; it still starts.
5. Check `application.properties` says `server.port=8080`.

---

## Step 1: Task 1, GroundMission (5 min)

File: `model/GroundMission.java`. Replace the two TODO bodies:

```java
@Override
public int energyUnits() {
    long energy = (2L * getDistanceKm()) + (5L * getLoadKg());
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

Don't touch the constructor, `isMedical()` or the fields.
**Checkpoint:** compiles (no red). For 101: 24 + 15 = 39.

---

## Step 2: Task 2.1 + 2.2, create AirMission (8 min)

IntelliJ: right-click package `model` → New → Java Class → `AirMission`.

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

    public int getWindPenalty() {
        return windPenalty;
    }

    public boolean isCritical() {
        return critical;
    }

    @Override
    public int energyUnits() {
        long energy = (3L * getDistanceKm())
                + (8L * getLoadKg())
                + (2L * windPenalty);
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

**Checkpoint:** no red. (IntelliJ shortcut: put cursor in a class that has red "must implement abstract methods" → Alt+Enter → Implement methods.)

---

## Step 3: Seed the repository (2 min)

File: `repository/MissionRepository.java`. In the constructor, replace the `// TODO after Task 2` comment with **one** line:

```java
public MissionRepository(){
    map.put(101L, new GroundMission(101, 12, 3, true));
    map.put(102L, new GroundMission(102, 4, 2, false));
    map.put(103L, new AirMission(103, 8, 1, 3, true));    // ← new
}
```

Don't duplicate 101/102. The starter's imports use `edu.practice.dispatch.model.*`, so `AirMission` is already visible.

**Checkpoint:** run the app. Open `http://localhost:8080/missions`. You should see three missions. (Air one shows `windPenalty` and `critical`.)

---

## Step 4: Task 3.1, `energy(id)` (3 min)

File: `service/DispatchService.java`.

```java
public int energy(long id) {
    DispatchMission mission = repo.find(id);
    if (mission == null) {
        throw new MissionNotFoundException(id);
    }
    return mission.energyUnits();
}
```

---

## Step 5: Task 3.2, `addAir` (10 min)

```java
public AirMission addAir(NewAirMission request) {
    if (request == null) {
        throw new IllegalArgumentException("Request body is required.");
    }
    checkRange("id", request.getId(), 1, 10000);
    checkRange("distanceKm", request.getDistanceKm(), 1, 10000);
    checkRange("loadKg", request.getLoadKg(), 0, 10000);
    checkRange("windPenalty", request.getWindPenalty(), 0, 10000);

    if (repo.exists(request.getId())) {
        throw new IllegalArgumentException("Mission ID " + request.getId() + " already exists.");
    }

    AirMission mission = new AirMission(
            request.getId(),
            request.getDistanceKm(),
            request.getLoadKg(),
            request.getWindPenalty(),
            request.getCritical());

    mission.energyUnits();      // calculate BEFORE saving: overflow would throw here
    repo.save(mission);
    return mission;
}

private void checkRange(String name, long value, long min, long max) {
    if (value < min || value > max) {
        throw new IllegalArgumentException(name + " must be between " + min + " and " + max + ".");
    }
}
```

Notes:
- **The starter's signature is `DispatchMission addAir(NewAirMission request)`.** Returning `AirMission` also compiles (covariant) and matches the paper ("created AirMission"), but you may keep the starter's return type if you prefer. Don't change the parameter.
- The helper `checkRange` avoids writing four near-identical `if` blocks. Writing them out longhand (as in the model solution) is equally correct, so use what you can type fastest.
- `int` arguments widen to `long` automatically when passed to `checkRange`.
- **Order = validate → duplicate → build → calculate → save → return.**

---

## Step 6: Task 2.3 + 3.3a, `plan` (15 min)

```java
public DispatchPlan plan(int capacity, boolean urgentOnly) {
    if (capacity <= 0) {
        throw new IllegalArgumentException("Capacity must be greater than zero.");
    }

    // 1) filter (filtered-out missions never reach either result list)
    List<DispatchMission> eligible = new ArrayList<>();
    for (DispatchMission m : repo.all()) {
        if (!urgentOnly || m.priority() >= 3) {
            eligible.add(m);
        }
    }

    // 2) sort: priority DESC, energy ASC, id ASC
    eligible.sort(Comparator.comparingInt(DispatchMission::priority).reversed()
            .thenComparingInt(DispatchMission::energyUnits)
            .thenComparingLong(DispatchMission::getId));

    // 3) greedy pass over the WHOLE list
    List<PlanLine> selected = new ArrayList<>();
    List<Long> skipped = new ArrayList<>();
    int used = 0;
    for (DispatchMission m : eligible) {
        int energy = m.energyUnits();
        if (energy <= capacity - used) {
            selected.add(new PlanLine(m.getId(), m.getClass().getSimpleName(), m.priority(), energy));
            used += energy;
        } else {
            skipped.add(m.getId());          // keep looping: later, cheaper missions may fit
        }
    }

    // 4) totals + return
    return new DispatchPlan(capacity, used, capacity - used, selected, skipped);
}
```

The starter has `import java.util.*;` and `import edu.practice.dispatch.model.*;` so `List`, `ArrayList`, `Comparator`, `Collection` all resolve. If a name is red, Alt+Enter → Import class.

**Why `!urgentOnly || m.priority() >= 3`?** If `urgentOnly` is false, everything is eligible (the `||` short-circuits). If true, only priority ≥ 3 pass.

**If you'd rather not use the chained comparator**, use the manual `Collections.sort(..., new Comparator<...>(){...})` version in `02_Concepts_Refresher.md` §7. Both give identical results.

**Checkpoint (can't test via HTTP yet; the controller comes next).** Re-read the code against the trace in `01_Decode_the_Paper.md` §3.3.

---

## Step 7: Task 3.3b, GlobalExceptionHandler (6 min)

File: `exception/GlobalExceptionHandler.java`. The starter already has `@RestControllerAdvice` and wildcard imports for `org.springframework.http.*` and `org.springframework.web.bind.annotation.*`. Add:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

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
}
```

Same package as `ErrorResponse` and `MissionNotFoundException`, so no extra imports.

**Safety zip now** (see Step 10), then continue.

---

## Step 8: Task 4, the three endpoints (10 min)

File: `controller/DispatchController.java`. Keep `all()`. Replace the TODO comments with:

```java
@GetMapping("/{id}/energy")
public Map<String, Integer> energy(@PathVariable("id") long id) {
    return Map.of("energyUnits", service.energy(id));
}

@PostMapping("/air")
@ResponseStatus(HttpStatus.CREATED)
public DispatchMission addAir(@RequestBody NewAirMission request) {
    return service.addAir(request);
}

@GetMapping("/plan")
public DispatchPlan plan(
        @RequestParam("capacity") int capacity,
        @RequestParam(name = "urgentOnly", defaultValue = "false") boolean urgentOnly) {
    return service.plan(capacity, urgentOnly);
}
```

The starter's imports are wildcards (`edu.practice.dispatch.model.*`, `java.util.*`, `org.springframework.web.bind.annotation.*`, `org.springframework.http.*`), so `Map`, `HttpStatus`, and all annotations resolve.

**Path trap:** class is `@RequestMapping("/missions")`, so write `"/air"`, `"/plan"`, `"/{id}/energy"`, never `"/missions/air"`.

**Route trap:** `GET /missions/plan` vs `GET /missions/{id}/energy` don't clash (different shape). 

---

## Step 9: Run Section B (10 min)

Restart the app (in-memory data resets). Open `section-b-checks.http` in IntelliJ (or use Swagger UI at `http://localhost:8080/swagger-ui.html`, or curl) and run **in order**:

| Check | Expect |
| :-- | :-- |
| B1 `GET /missions/101/energy` | `{"energyUnits":39}` |
| B2 `GET /missions/103/energy` | `{"energyUnits":38}` |
| B3 plan capacity 60 | selected ids 103, 102; skipped `[101]`; used 56; remaining 4 |
| B4 plan capacity 60 urgentOnly=true | selected 103; skipped `[101]`; used 38; remaining 22 |
| B5 capacity=0 → 400 `BAD_REQUEST`; POST duplicate 103 → 400; POST distanceKm 0 → 400; GET 999 → 404 `NOT_FOUND` | status codes **and** `code` text |
| B6 POST 104 → **201**; GET /missions has 104; plan capacity 125 | selected `[103,101,104,102]`; used 125; remaining 0 |

Priority 3 for mission 101 (B1) appears as `"priority":3` inside a `selected` PlanLine. Capacity 60 doesn't select 101, so also try `capacity=200` to see 101's PlanLine.

Expected JSON for B3 (key order may differ):

```json
{
  "capacity": 60, "energyUsed": 56, "energyRemaining": 4,
  "selected": [
    {"id":103,"type":"AirMission","priority":5,"energyUnits":38},
    {"id":102,"type":"GroundMission","priority":1,"energyUnits":18}
  ],
  "skipped": [101]
}
```

---

## Step 10: Safety zip and submission (the last 10 minutes of the exam)

- Make a safety ZIP after **Task 3** and after **Task 4**, and upload if the exam system allows.
- Final: close nothing, but **Stop > clean build**; then zip the **whole project folder** (the one containing `pom.xml`, `src/`, and any `test/`). Never just `.java` files.
- **Open the ZIP** (or extract it elsewhere) and check that `AirMission.java` is inside and `pom.xml` is at the top level of the project folder.
- Delete the `target/` folder first if the ZIP is huge (it's rebuildable). Keep `src/` and `pom.xml`.

---

## Timing guide for the real 100 minutes

| Clock | Do |
| :-- | :-- |
| 0:00-0:12 | Read paper (5-step routine) + read every starter file. Write test data on paper |
| 0:12-0:20 | Task 1 + Task 2.1 + 2.2 + repository seed (everything model-side). Run once |
| 0:20-0:40 | `plan` + `energy` + `addAir` in the service |
| 0:40-0:50 | Handler + controller |
| 0:50-1:10 | Section B runs, fix failures |
| 1:10-1:25 | Re-read the paper's bullet points one by one; tick each off |
| 1:25-1:40 | Zip, extract-check, submit. Don't start new things |

**Stuck >10 minutes on something?** Leave a compiling stub, move on, return at the end.
