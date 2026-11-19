# M1: The Mock Paper, A to Z (Emergency Dispatch Energy Planner)

**For:** someone who has never done one of these papers before.
**Uses:** `MODERATE_Mock_Practice_Paper.docx`, the official starter (`Mock_Starter_Project_Java25_SpringBoot4.1.1_With_Swagger_UI_.zip`) and your lecturer's worked solution (`Mock_Lab_Based_Practical_Solution_Code.zip`).
**Goal:** understand every file, every task and every line well enough to write the whole solution from a blank starter in under 100 minutes.

> I compiled your lecturer's solution and ran every check in the paper against it: 39 / 38 energy, the capacity-60 plan, the urgent-only plan, 400 and 404 errors, and the capacity-125 plan after POST 104. All pass. The code in this guide follows **your lecturer's style** (plain classes, loops, `Collections.sort` with a `Comparator`), because that is what the markers wrote.

---

## Contents

1. Can you get 60+? (honest answer and the mark map)
2. Your timetable until the exam
3. Setting up and opening the project
4. The big picture: how a request flows
5. Tour of every starter file
6. Task 1: `GroundMission` (18)
7. Task 2: `AirMission` and the dispatch plan (22)
8. Task 3: lookup, validation, errors (20)
9. Task 4: the three endpoints (20)
10. Section B: testing in Swagger UI (20)
11. Zipping and submitting
12. Errors you may see and how to fix them
13. The eight patterns to memorise
14. The blank-page drill

---

## 1. Can you get 60+ in 3 days?

**Yes, realistically, if you practise this exact paper type by typing it, not just reading it.** The papers are very formulaic: the same seven or eight code patterns appear every time with different names. You do not need to "master all of OOP" to pass. You need to master **these patterns**.

Where the marks come from, easiest first:

| Part | Marks | How hard | What it needs |
| :-- | :-- | :-- | :-- |
| Task 1: two methods in an existing class | 18 | **Easy** | One formula, one `if` |
| Task 4: three controller methods | 20 | **Easy** (pure pattern) | Annotations you memorise |
| Section B: the app running and answering correctly | 20 | Easy **if** it starts and Tasks 1, 2.1, 3, 4 work | Restart + test in Swagger |
| Task 3: lookup, validation, error handler | 20 | Medium (pattern) | `if (x == null) throw`, range checks, two handler methods |
| Task 2.1 + 2.2: new subclass | 14 | Medium | Copy the shape of `GroundMission` |
| Task 2.3: sort + greedy plan | 8 | **Hardest** | Comparator + loop |

**The safe 60+ route:** Task 1 (18) + Task 2.1/2.2 (14) + Task 3 (20) + Task 4 (20) = 72 Section A marks available **without** the plan algorithm, plus most of Section B. Even with some mistakes you clear 60. The plan algorithm (Task 2.3) is the bonus you learn last.

**Biggest risk:** a project that does not start. Then Section B scores 0. Rule: **restart and check after every task; never leave half-typed code.**

---

## 2. Your timetable

Assumes the exam is on **Thursday**. If it is earlier, keep the order and squeeze the times. Each block: phone away, IntelliJ open, typing.

### Monday (today, from about 13:30)

| Time | Do | Use |
| :-- | :-- | :-- |
| 13:30–14:30 | Install/check Java 25, IntelliJ, open the mock starter, run it, open Swagger (§3). | This guide §3 |
| 14:30–16:00 | Read §4–§5: understand every starter file. Reformat each file (Ctrl+Alt+L) and read it aloud. | §4–§5 |
| 16:00–16:30 | Break, food. | |
| 16:30–18:30 | Tasks 1 and 4 with this guide open. Run and test each in Swagger. | §6, §9 |
| 18:30–19:30 | Break. | |
| 19:30–21:30 | Tasks 3 and 2.1/2.2 with the guide open. Test 400/404. | §7.1–7.2, §8 |
| 21:30–22:30 | Read Task 2.3 (§7.3) slowly, trace the tables on paper. Sleep by 23:30. | §7.3 |

### Tuesday

| Time | Do |
| :-- | :-- |
| 09:00–10:30 | Type Task 2.3 yourself with the guide. Check capacity 60, urgent, 125. |
| 10:45–12:25 | **Mock attempt #1 from a fresh starter**, 100-minute timer, guide allowed but try not to look. |
| 12:25–13:30 | Compare with the staff solution. Write down every mistake. Lunch. |
| 14:00–15:30 | OOP refresh: Phase 4 (inheritance) §2–3 and Phase 5 (polymorphism) §2–4 cheat sheets. S4 (error handling) cheat sheet. |
| 15:45–17:45 | **Mock attempt #2**, fresh starter, **no guide**, 100 minutes. |
| 18:00–19:00 | Mark yourself against §10's Section B table. Fix gaps. |
| 20:00–22:00 | Read M2 (advanced paper) Parts 1–3: the new patterns (409 conflict, DELETE, two constraints). |

### Wednesday

| Time | Do |
| :-- | :-- |
| 09:00–10:40 | **Advanced paper attempt**, fresh starter, 100 minutes. |
| 10:40–12:00 | Compare with the M2 solution; learn what was new. |
| 13:00–14:40 | **Mock attempt #3**, no guide, aim for under 75 minutes. |
| 15:00–16:40 | The other practice papers (SIMPLE parking, then MODERATE dispatch from yesterday's pack), 100 minutes for one of them. Note: those use records; the real exam style uses classes. Same logic. |
| 17:00–18:00 | §13 memory sheet: write all eight patterns from memory on paper. |
| 19:00–20:30 | Light: re-read §12 (errors) and the Phase 8 / S6 checklists. Practise zipping. |
| 21:00 | Stop. Sleep early. |

### Thursday (exam day)

| Time | Do |
| :-- | :-- |
| Morning | 30 minutes: write the eight patterns from memory once. No new material. |
| In the exam | Follow §2 of S6: 5 min setup, 10 min reading, then Task 1 → 4 → 3 → 2 (or in order), safety ZIP after Task 3, stop coding 10 minutes before the end. |

---

## 3. Setting up and opening the project

### 3.1 What the lab gives you

| Tool | Version | What it is |
| :-- | :-- | :-- |
| **JDK** (Java Development Kit) | **25** | Java itself: the compiler and the JVM (OOP Pre-Phase 0). |
| **Maven** | 3.9+ | Downloads libraries listed in `pom.xml`, builds and runs the app (S1 §4). |
| **Spring Boot** | **4.1.1** | The framework (S1 §3). Fixed by `pom.xml`: never change it. |
| **IntelliJ IDEA** | any recent | The editor. |
| **springdoc** | 3.2.0 | Generates **Swagger UI** (S1 §10). Already in this `pom.xml`. |

### 3.2 Open and run, step by step

1. Unzip `Mock_Starter_Project_...zip`. You get a folder `drone-practice-starter`.
2. IntelliJ → **File → Open** → select `drone-practice-starter` (the folder that **contains `pom.xml`**) → **Open** → **Trust Project**.
3. **File → Project Structure → Project → SDK**: choose a **25** JDK. (If none is listed: *Add SDK → Download JDK → version 25*, or pick the lab's installed one.)
4. Wait for the Maven progress bar at the bottom to finish. If code stays red: Maven tool window (right edge, "m" icon) → **Reload All Maven Projects** (circular arrows).
5. Open `src/main/java/edu/practice/dispatch/DispatchApplication.java` → green triangle beside `main` → **Run**.
6. Console should end with `Tomcat started on port 8080` and `Started DispatchApplication`.
7. Browser: `http://localhost:8080/missions` → JSON for missions 101 and 102.
8. Browser: `http://localhost:8080/swagger-ui.html` → the Swagger page listing `GET /missions`.

> **Port confusion in the starter.** The starter's `README.md` says port **8082**, but `src/main/resources/application.properties` says `server.port=8080`, and the paper and the solution guide both say **8080**. The **properties file wins**: Spring reads it, not the README. Always trust `application.properties`.

### 3.3 Make the code readable first

The starter is written in a squashed style, several statements per line. Your first action in each file: **Ctrl + Alt + L** (Mac: Cmd + Option + L) = **Reformat Code**. It changes only spacing, never meaning, and makes everything easy to read. Do it for every file in the first 10 minutes of the exam.

---

## 4. The big picture

### 4.1 The packages

```
edu.practice.dispatch
├── DispatchApplication        main class: starts Spring
├── controller/                HTTP: URLs in, JSON out          (Task 4)
│   └── DispatchController
├── service/                   rules: calculations, validation (Tasks 2.3, 3)
│   └── DispatchService
├── repository/                storage: a Map of missions
│   └── MissionRepository
├── model/                     the OOP classes and data shapes (Tasks 1, 2)
│   ├── DispatchMission        abstract parent (provided)
│   ├── GroundMission          subclass (Task 1)
│   ├── AirMission             subclass YOU CREATE (Task 2)
│   ├── NewAirMission          request body shape (provided)
│   ├── PlanLine               one selected mission in a plan (provided)
│   └── DispatchPlan           the whole plan response (provided)
└── exception/                 errors (Task 3)
    ├── MissionNotFoundException   (provided)
    ├── ErrorResponse              error JSON shape (provided)
    └── GlobalExceptionHandler     YOU COMPLETE
```

### 4.2 One request, start to finish

`GET /missions/101/energy`:

```
Browser/Swagger
   │  GET /missions/101/energy
   ▼
DispatchController.energy(101)              reads the 101 from the URL
   │  service.energy(101)
   ▼
DispatchService.energy(101)                 finds the mission, or throws
   │  repo.find(101)
   ▼
MissionRepository.find(101)                 map.get(101) → the GroundMission object
   ▲
   │  mission.energyUnits()  → 39           POLYMORPHISM: GroundMission's formula runs
   │
DispatchController returns Map.of("energyUnits", 39)
   ▼
Spring + Jackson → {"energyUnits":39}, status 200
```

If 999 is asked for: `repo.find(999)` returns `null` → the service throws `MissionNotFoundException` → it travels up past the controller → `GlobalExceptionHandler` catches it → `404` with `{"code":"NOT_FOUND","message":"Mission not found: 999"}`.

---

## 5. Tour of every starter file

Read each one with the explanation beside it. Words in **bold** are defined in the earlier guides (Phase / S numbers given).

### 5.1 `DispatchApplication.java`

```java
@SpringBootApplication
public class DispatchApplication {
    public static void main(String[] args) {
        SpringApplication.run(DispatchApplication.class, args);
    }
}
```

Starts Spring: scans the packages below `edu.practice.dispatch`, creates one object of each labelled class, wires them, starts Tomcat on port 8080 (S1 §6). **Never edit.**

### 5.2 `model/DispatchMission.java` (the abstract parent: read carefully)

Reformatted:

```java
public abstract class DispatchMission {

    private final long id;
    private final int distanceKm;
    private final int loadKg;

    protected DispatchMission(long id, int distanceKm, int loadKg) {
        this.id = id;
        this.distanceKm = distanceKm;
        this.loadKg = loadKg;
    }

    public long getId() { return id; }
    public int getDistanceKm() { return distanceKm; }
    public int getLoadKg() { return loadKg; }

    public abstract int energyUnits();
    public abstract int priority();
}
```

| Line | Meaning | Consequence for you |
| :-- | :-- | :-- |
| `public abstract class` | Cannot be created with `new` (Phase 5 §4) | You always work with `GroundMission` / `AirMission` objects |
| `private final long id` (and the other two) | Hidden and set once | In subclasses use `getId()`, `getDistanceKm()`, `getLoadKg()`. Writing `distanceKm` directly in a subclass = compile error "has private access" |
| `protected DispatchMission(long id, int distanceKm, int loadKg)` | Only subclasses may call it, via `super(...)` | Your `AirMission` constructor's first line: `super(id, distanceKm, loadKg);` in **this order** |
| `public abstract int energyUnits();` | Every subclass must write its own version | Override with **exactly** `public int energyUnits()` |
| `public abstract int priority();` | Same | Override with exactly `public int priority()` |

### 5.3 `model/GroundMission.java` (Task 1)

```java
public class GroundMission extends DispatchMission {

    private final boolean medical;

    public GroundMission(long id, int distanceKm, int loadKg, boolean medical) {
        super(id, distanceKm, loadKg);
        this.medical = medical;
    }

    public boolean isMedical() { return medical; }

    @Override
    public int energyUnits() {
        // TODO Task 1: energy = 2*distanceKm + 5*loadKg; avoid integer overflow.
        throw new UnsupportedOperationException("Task 1");
    }

    @Override
    public int priority() {   // TODO Task 1: medical=3, otherwise=1.
        throw new UnsupportedOperationException("Task 1");
    }
}
```

This is your **template** for `AirMission`: same shape, different fields and formulas.

`throw new UnsupportedOperationException("Task 1");` is a placeholder: it compiles, so the app starts, but crashes **if called**. You replace it.

### 5.4 `model/NewAirMission.java` (the POST request body)

A plain class: private fields `id`, `distanceKm`, `loadKg`, `windPenalty`, `critical`; a **no-arg constructor** (Jackson needs it: S2 §4.3); a full constructor; getters and setters.

Note the boolean getter is **`getCritical()`**, not `isCritical()`. So in the service you write `request.getCritical()`. (Jackson accepts both styles, so `"critical": true` in JSON still fills it.)

**`NewAirMission` vs `AirMission`** (the solution guide stresses this):

| | `NewAirMission` | `AirMission` |
| :-- | :-- | :-- |
| What | The **request data** the client sends | The **real mission object** with behaviour |
| Created by | Jackson, from JSON | **Your service**, from the request |
| Has formulas? | No | Yes (`energyUnits`, `priority`) |
| Extends `DispatchMission`? | No | Yes |

### 5.5 `model/PlanLine.java` and `model/DispatchPlan.java` (response shapes)

`PlanLine(long id, String type, int priority, int energyUnits)`: one selected mission in the plan.
`DispatchPlan(int capacity, int energyUsed, int energyRemaining, List<PlanLine> selected, List<Long> skipped)`: the whole answer.

Both are plain classes with no-arg + full constructors and getters. You only **create** them with the full constructor: `new PlanLine(...)`, `new DispatchPlan(...)`. Their getters make the JSON keys `capacity`, `energyUsed`, `energyRemaining`, `selected`, `skipped`.

### 5.6 `repository/MissionRepository.java`

```java
@Repository
public class MissionRepository {

    private final Map<Long, DispatchMission> map = new ConcurrentHashMap<>();

    public MissionRepository() {
        map.put(101L, new GroundMission(101, 12, 3, true));
        map.put(102L, new GroundMission(102, 4, 2, false));
        // TODO after Task 2: seed AirMission(103,8,1,3,true).
    }

    public Collection<DispatchMission> all() { return List.copyOf(map.values()); }
    public DispatchMission find(long id)     { return map.get(id); }
    public boolean exists(long id)           { return map.containsKey(id); }
    public void save(DispatchMission m)      { map.put(m.getId(), m); }
}
```

| Method | Returns | Use it for |
| :-- | :-- | :-- |
| `all()` | Every mission (as the parent type) | Listing and planning |
| `find(id)` | The mission, **or `null`** if absent | Lookup; **you** must check for `null` |
| `exists(id)` | `true`/`false` | Duplicate-id check |
| `save(m)` | nothing | Storing a new mission |

`ConcurrentHashMap` = a thread-safe `HashMap` (S5 §3.2). Keys are `Long` ids. Only the seed comment line may be edited. Data lives in memory: **restarting the app resets it.**

### 5.7 `service/DispatchService.java`

```java
@Service
public class DispatchService {

    private final MissionRepository repo;

    public DispatchService(MissionRepository repo) { this.repo = repo; }   // constructor injection

    public Collection<DispatchMission> all() { return repo.all(); }

    public int energy(long id)                       { /* TODO Task 3 */ }
    public DispatchMission addAir(NewAirMission req) { /* TODO Task 3 */ }
    public DispatchPlan plan(int capacity, boolean urgentOnly) { /* TODO Task 2 */ }
}
```

Spring builds the repository, then calls this constructor with it (S5 §2). You fill the three methods.

### 5.8 `controller/DispatchController.java`

```java
@RestController
@RequestMapping("/missions")
public class DispatchController {

    private final DispatchService service;

    public DispatchController(DispatchService service) { this.service = service; }

    @GetMapping
    public Collection<DispatchMission> all() { return service.all(); }

    // TODO Task 4: GET /{id}/energy; POST /air (HTTP 201);
    // GET /plan?capacity=...&urgentOnly=false; delegate all logic to service.
}
```

Every URL starts with `/missions` (class-level `@RequestMapping`). Keep `all()`.

### 5.9 `exception/` files

- `MissionNotFoundException(long id)`: `super("Mission not found: " + id)` sets the message. Provided.
- `ErrorResponse(String code, String message)`: the error JSON `{"code":..., "message":...}`. Provided; has getters.
- `GlobalExceptionHandler`: `@RestControllerAdvice`, empty. You add two methods (Task 3.3).

---

## 6. Task 1: `GroundMission` (18 marks)

### 6.1 Task 1.1: `energyUnits()` (12)

Formula: **2 × distanceKm + 5 × loadKg**, using `long` maths and a safe conversion to `int`.

```java
@Override
public int energyUnits() {
    long energy = (2L * getDistanceKm()) + (5L * getLoadKg());
    return Math.toIntExact(energy);
}
```

Word by word:

| Piece | Meaning |
| :-- | :-- |
| `@Override` | "This replaces the parent's abstract method." The compiler checks the name and parameters match (Phase 5 §2.3). |
| `public int energyUnits()` | Exactly the parent's header. |
| `long energy` | A 64-bit whole number: huge range, so the sum cannot overflow here. |
| `2L` | The number 2 **as a long**. Makes `2L * getDistanceKm()` happen in `long`. Writing `2 *` would multiply in `int` first, which could overflow **before** reaching the `long` variable. |
| `getDistanceKm()` | The parent's getter (the field is private to the parent). |
| `Math.toIntExact(energy)` | Convert `long` → `int`; **throws `ArithmeticException`** if it does not fit, instead of silently producing a wrong number. That is "do not allow silent integer overflow". |

Check: 101 → 2×12 + 5×3 = 24 + 15 = **39**.

### 6.2 Task 1.2: `priority()` (6)

```java
@Override
public int priority() {
    if (medical) {
        return 3;
    }
    return 1;
}
```

`medical` is this class's own field, so it can be used directly. If `true`, return 3 and stop; otherwise the method continues to `return 1`. (One-line alternative: `return medical ? 3 : 1;`, the conditional operator from S6 §B2.)

**Test now:** restart; nothing visible changes yet (no endpoint calls it). That is fine.

---

## 7. Task 2: `AirMission` and dispatch planning (22 marks)

### 7.1 Task 2.1: create the class (7)

**Steps:** in the Project panel right-click `model` → **New → Java Class** → type `AirMission` → Enter. IntelliJ writes `package edu.practice.dispatch.model;` for you.

```java
package edu.practice.dispatch.model;

public class AirMission extends DispatchMission {

    private final int windPenalty;
    private final boolean critical;

    public AirMission(long id, int distanceKm, int loadKg,
                      int windPenalty, boolean critical) {
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
}
```

| Line | Why |
| :-- | :-- |
| `extends DispatchMission` | IS-A mission: goes into the same map, list and plan. |
| `private final` fields | Same style as `GroundMission`'s `medical`. |
| Parameter order `(id, distanceKm, loadKg, windPenalty, critical)` | Must match the seed in the paper: `AirMission(103, 8, 1, 3, true)`. |
| `super(id, distanceKm, loadKg);` first | The parent's fields are private and final: only its constructor can set them (Phase 4 §3). |
| `this.windPenalty = windPenalty;` | Store the subclass's own values. |
| Getters | "appropriate getters"; they also put `windPenalty` and `critical` into the JSON. |

At this point the class is **red**: "AirMission is not abstract and does not override abstract method priority()". That is Task 2.2.

### 7.2 Task 2.2: the calculations (7)

Add inside `AirMission`:

```java
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
```

Check: 103 → 3×8 + 8×1 + 2×3 = 24 + 8 + 6 = **38**; critical → **5**.

**Seed it:** in `MissionRepository`'s constructor, replace the TODO comment with:

```java
map.put(103L, new AirMission(103, 8, 1, 3, true));
```

(IntelliJ may need the import: click `AirMission`, **Alt + Enter → Import class**. The starter's `import edu.practice.dispatch.model.*;` already covers it.)

**Test now:** restart → `GET /missions` shows three missions, 103 with `windPenalty` and `critical`.

### 7.3 Task 2.3: `plan(capacity, urgentOnly)` (8)

The rules, from the paper:

1. If `urgentOnly`, keep only missions with priority ≥ 3. Filtered missions appear in **neither** list.
2. Sort: **priority descending**, then **energy ascending**, then **id ascending**.
3. Walk the **whole** sorted list. If a mission fits the remaining capacity: select it. If not: add its id to `skipped` **and keep going**.
4. No `instanceof`: only call `priority()`, `energyUnits()`, `getId()` through `DispatchMission`.
5. Return `new DispatchPlan(capacity, energyUsed, energyRemaining, selected, skipped)`.

Plus Task 3.3: capacity ≤ 0 → `IllegalArgumentException`.

#### Step A: validate and filter

```java
public DispatchPlan plan(int capacity, boolean urgentOnly) {
    if (capacity <= 0) {
        throw new IllegalArgumentException("Capacity must be greater than zero.");
    }

    List<DispatchMission> eligible = new ArrayList<>();
    for (DispatchMission mission : repo.all()) {
        if (!urgentOnly || mission.priority() >= 3) {
            eligible.add(mission);
        }
    }
```

| Piece | Meaning |
| :-- | :-- |
| `List<DispatchMission> eligible = new ArrayList<>();` | An empty, changeable list (Phase 3). We need our own list because `repo.all()` is read-only and cannot be sorted. |
| `for (DispatchMission mission : repo.all())` | "For each mission in the repository." |
| `!urgentOnly \|\| mission.priority() >= 3` | Keep it if we are **not** filtering, **or** it is urgent. Truth table below. |

| urgentOnly | priority | `!urgentOnly` | `priority >= 3` | Kept? |
| :-- | :-- | :-- | :-- | :-- |
| false | any | true | (not needed) | yes |
| true | 5 | false | true | yes |
| true | 1 | false | false | **no** |

#### Step B: sort with a `Comparator`

A **`Comparator`** decides which of two objects comes first. Its `compare(first, second)` method returns:

| Return value | Meaning |
| :-- | :-- |
| negative | `first` goes **before** `second` |
| zero | equal (try the next rule) |
| positive | `first` goes **after** `second` |

`Integer.compare(a, b)` returns exactly that for two numbers: negative if `a < b`, zero if equal, positive if `a > b`. So:

- `Integer.compare(first.x, second.x)` → **ascending** (small first).
- `Integer.compare(second.x, first.x)` → **descending** (arguments swapped: big first).

Your lecturer's code:

```java
    Collections.sort(eligible, new Comparator<DispatchMission>() {
        @Override
        public int compare(DispatchMission first, DispatchMission second) {
            int result = Integer.compare(second.priority(), first.priority());    // priority DESC
            if (result != 0) {
                return result;
            }
            result = Integer.compare(first.energyUnits(), second.energyUnits());  // energy ASC
            if (result != 0) {
                return result;
            }
            return Long.compare(first.getId(), second.getId());                    // id ASC
        }
    });
```

| Piece | Meaning |
| :-- | :-- |
| `Collections.sort(list, comparator)` | Sorts the list **in place** using the comparator. (`import java.util.Collections;`) |
| `new Comparator<DispatchMission>() { ... }` | An **anonymous class**: a class with no name, written on the spot, that implements the `Comparator` interface (Phase 5 §5). It has one method, `compare`. |
| `second.priority(), first.priority()` | Swapped order = **descending**. |
| `if (result != 0) return result;` | If this rule already decides, stop. Otherwise fall to the next rule (the tie-breaker). |
| `Long.compare` | Same as `Integer.compare` but for `long` ids. |

**Shorter, equally correct alternative** (lambda + chained comparator, S6 §B5): 

```java
    eligible.sort(Comparator.comparingInt(DispatchMission::priority).reversed()
            .thenComparingInt(DispatchMission::energyUnits)
            .thenComparingLong(DispatchMission::getId));
```

Use whichever you can write correctly **from memory**. The long version is easier to reason about; the short one has the `reversed()` trap (S6 §B5.2).

#### Step C: the greedy loop

```java
    List<PlanLine> selected = new ArrayList<>();
    List<Long> skipped = new ArrayList<>();
    int energyUsed = 0;

    for (DispatchMission mission : eligible) {
        int energy = mission.energyUnits();
        if (energy <= capacity - energyUsed) {
            selected.add(new PlanLine(
                    mission.getId(),
                    mission.getClass().getSimpleName(),
                    mission.priority(),
                    energy));
            energyUsed = energyUsed + energy;
        } else {
            skipped.add(mission.getId());
        }
    }

    int energyRemaining = capacity - energyUsed;
    return new DispatchPlan(capacity, energyUsed, energyRemaining, selected, skipped);
}
```

| Piece | Meaning |
| :-- | :-- |
| `capacity - energyUsed` | What is left right now. |
| `energy <= ...` | "Does it fit?" |
| `mission.getClass().getSimpleName()` | The real class's short name, `"AirMission"` or `"GroundMission"`, for the `type` field. A label, not a branch, so it respects the no-type-check rule (S3 §8.1). |
| `else skipped.add(...)` | Does not fit: remember it **and continue** (no `break`). |
| `List<Long> skipped` + `skipped.add(mission.getId())` | `getId()` returns `long`; Java **autoboxes** it to `Long` for the list (Phase 3 §4). |

#### Traces (do these on paper once)

Energies: 101 = 39 (p3), 102 = 2×4 + 5×2 = **18** (p1), 103 = 38 (p5).

**Sorted:** 103 (p5) → 101 (p3) → 102 (p1).

**Capacity 60, urgentOnly=false:**

| Mission | Energy | Left before | Fits? | Left after |
| :-- | :-- | :-- | :-- | :-- |
| 103 | 38 | 60 | yes → selected | 22 |
| 101 | 39 | 22 | **no → skipped** | 22 |
| 102 | 18 | 22 | yes → selected | 4 |

→ selected [103, 102], skipped [101], used 56, remaining 4. ✓ (B3)

**Capacity 60, urgentOnly=true:** 102 filtered out first. 103 selected (22 left), 101 skipped. → selected [103], skipped [101]. ✓ (B4)

**After POST 104** (energy 3×6 + 8×1 + 2×2 = 30, p2): sorted 103 (p5), 101 (p3), 104 (p2), 102 (p1). **Capacity 125:** 38 → 87 left; 39 → 48; 30 → 18; 18 → 0. → selected [103, 101, 104, 102], used 125, remaining 0. ✓ (B6)

---

## 8. Task 3: lookup, validation, errors (20 marks)

### 8.1 Task 3.1: `energy(id)` (6)

```java
public int energy(long id) {
    DispatchMission mission = repo.find(id);
    if (mission == null) {
        throw new MissionNotFoundException(id);
    }
    return mission.energyUnits();
}
```

`repo.find` returns `null` when the id is unknown (§5.6). `mission.energyUnits()` is **polymorphic**: whichever subclass the object is, its own formula runs.

### 8.2 Task 3.2: `addAir(request)` (7)

```java
public DispatchMission addAir(NewAirMission request) {
    if (request == null) {
        throw new IllegalArgumentException("Request body is required.");
    }
    validateAirMission(request);

    if (repo.exists(request.getId())) {
        throw new IllegalArgumentException(
                "A mission with ID " + request.getId() + " already exists.");
    }

    AirMission mission = new AirMission(
            request.getId(),
            request.getDistanceKm(),
            request.getLoadKg(),
            request.getWindPenalty(),
            request.getCritical());

    mission.energyUnits();          // calculate BEFORE saving
    repo.save(mission);
    return mission;
}

private void validateAirMission(NewAirMission request) {
    if (request.getId() < 1 || request.getId() > 10000) {
        throw new IllegalArgumentException("ID must be between 1 and 10000.");
    }
    if (request.getDistanceKm() < 1 || request.getDistanceKm() > 10000) {
        throw new IllegalArgumentException("distanceKm must be between 1 and 10000.");
    }
    if (request.getLoadKg() < 0 || request.getLoadKg() > 10000) {
        throw new IllegalArgumentException("loadKg must be between 0 and 10000.");
    }
    if (request.getWindPenalty() < 0 || request.getWindPenalty() > 10000) {
        throw new IllegalArgumentException("windPenalty must be between 0 and 10000.");
    }
}
```

| Piece | Why |
| :-- | :-- |
| Range check pattern `x < min \|\| x > max` | "Outside the range" = below the minimum **or** above the maximum. Copy this pattern for every field. |
| A `private` helper method | Keeps `addAir` short. Private = only this class uses it. Allowed: the paper only fixes the public methods. |
| Duplicate check with `repo.exists(...)` | `save` would silently **replace** mission 101 otherwise. |
| `mission.energyUnits();` before `save` | "Calculate the energy before saving." If it ever overflowed, `Math.toIntExact` would throw **before** anything is stored. |
| Every check **before** `repo.save` | "Do not save an invalid mission." |
| Return the `AirMission` | The controller returns it with 201. |

**About the overflow exception.** `Math.toIntExact` throws `ArithmeticException`, which the handler does not map (it would be a 500). With the paper's limits (≤ 10,000) the largest possible energy is 3×10,000 + 8×10,000 + 2×10,000 = 130,000, which fits easily, so your lecturer's solution does not catch it. If you want to be extra safe, wrap it:

```java
try {
    mission.energyUnits();
} catch (ArithmeticException e) {
    throw new IllegalArgumentException("Energy value is too large.");
}
```

### 8.3 Task 3.3: capacity check + handler (7)

The capacity check is the first `if` in `plan` (§7.3 Step A).

`GlobalExceptionHandler`:

```java
package edu.practice.dispatch.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MissionNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleMissionNotFound(MissionNotFoundException exception) {
        ErrorResponse error = new ErrorResponse("NOT_FOUND", exception.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleInvalidInput(IllegalArgumentException exception) {
        ErrorResponse error = new ErrorResponse("BAD_REQUEST", exception.getMessage());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
}
```

| Piece | Meaning (S4) |
| :-- | :-- |
| `@RestControllerAdvice` | Watches every controller for exceptions. Your lecturer's solution uses `@ControllerAdvice`; with `ResponseEntity` both work. **Keep what the starter has.** |
| `@ExceptionHandler(X.class)` | "Run this method when an X escapes a controller." |
| Parameter `X exception` | Spring passes the thrown object in. |
| `exception.getMessage()` | The text set in the exception's constructor: the "descriptive message". |
| `"NOT_FOUND"` / `"BAD_REQUEST"` | **Exactly** these codes, as the paper says. |
| `ResponseEntity.status(...).body(...)` | Choose the status and the JSON body together. |

The starter already imports `org.springframework.http.*` and `org.springframework.web.bind.annotation.*`, which cover everything above.

---

## 9. Task 4: the three endpoints (20 marks)

Add inside `DispatchController`, under `all()`:

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

### 9.1 GET `/missions/{id}/energy` (5)

| Piece | Meaning |
| :-- | :-- |
| `@GetMapping("/{id}/energy")` | Answers GET `/missions/{id}/energy`; `{id}` is a placeholder. |
| `@PathVariable("id") long id` | Copy the `{id}` part into `id`. (`("id")` names it explicitly; works either way.) |
| `Map<String, Integer>` + `Map.of("energyUnits", ...)` | Builds `{"energyUnits":39}` exactly as the paper's example (S6 §B6). |

### 9.2 POST `/missions/air` (5)

| Piece | Meaning |
| :-- | :-- |
| `@PostMapping("/air")` | Answers POST `/missions/air`. |
| `@ResponseStatus(HttpStatus.CREATED)` | **201 Created**, as required. Without it you get 200 and lose marks. |
| `@RequestBody NewAirMission request` | Jackson builds a `NewAirMission` from the JSON (no-arg constructor + setters). |
| Return type `DispatchMission` | The service returns the `AirMission`; Jackson writes all its getters. |

### 9.3 GET `/missions/plan` (10)

| Piece | Meaning |
| :-- | :-- |
| `@RequestParam("capacity") int capacity` | Reads `?capacity=60`. Required: missing → 400. |
| `@RequestParam(name = "urgentOnly", defaultValue = "false") boolean urgentOnly` | Reads `&urgentOnly=true`; **defaults to false** when left out (the paper's rule). |
| One line delegating to the service | "Keep controller methods focused on handling HTTP requests." |

Imports needed: `java.util.Map` (the starter's `import java.util.*;` covers it), `org.springframework.http.HttpStatus` (covered by `org.springframework.http.*`).

---

## 10. Section B: testing in Swagger UI (20 marks)

**Restart the app first.** Open `http://localhost:8080/swagger-ui.html`.

How to use Swagger for each request:

1. Click the endpoint row (e.g. `GET /missions/{id}/energy`) to expand it.
2. Click **Try it out**.
3. Fill the boxes (`id`, `capacity`, `urgentOnly`), or for POST edit the JSON in the **Request body** box.
4. Click **Execute**.
5. Scroll to **Server response**: read the **Code** (200, 201, 400, 404) and the **Response body**.

| Ref | Do | Expect |
| :-- | :-- | :-- |
| **B1** (3) | GET `/missions/101/energy` | 200, `{"energyUnits":39}`; and in a plan, 101 shows `"priority":3` |
| **B2** (3) | GET `/missions/103/energy` | `{"energyUnits":38}`; priority 5 in a plan |
| **B3** (3) | GET `/missions/plan` capacity 60, urgentOnly false | selected ids 103, 102; skipped `[101]`; `energyUsed` 56, `energyRemaining` 4 |
| **B4** (3) | Same with urgentOnly true | selected 103; skipped `[101]`; 102 in neither |
| **B5** (4) | capacity 0 → **400** `BAD_REQUEST`; POST with `"id":101` (duplicate) → **400**; POST with `"distanceKm":0` → **400**; GET `/missions/999/energy` → **404** `NOT_FOUND` | Status codes and codes exactly |
| **B6** (4) | POST `{"id":104,"distanceKm":6,"loadKg":1,"windPenalty":2,"critical":false}` → **201**; GET `/missions` includes 104; plan capacity 125 → selected 103, 101, 104, 102; used 125; remaining 0 | All three in **one run** (restart resets data) |

You can also use an IntelliJ `.http` file (S2 §11.3) with port **8080** and paths without `/api`.

---

## 11. Zipping and submitting

1. Stop the app (red square). **File → Save All**.
2. Right-click the project in IntelliJ → **Open In → Explorer** (Mac: Finder). Go **up one level**.
3. Right-click `drone-practice-starter` → **Compress / Send to → Compressed (zipped) folder**.
4. Open the ZIP: `pom.xml`, `src/main/java/.../model/AirMission.java` must be inside.
5. Upload it; follow the exam's naming rule.

---

## 12. Errors you may see

| Message | Cause | Fix |
| :-- | :-- | :-- |
| `AirMission is not abstract and does not override abstract method priority()` | A required override is missing | Add both `@Override` methods |
| `method does not override or implement a method from a supertype` | Typo in an override's name/parameters | Match the parent exactly |
| `distanceKm has private access in DispatchMission` | Used the parent's field directly | `getDistanceKm()` |
| `constructor DispatchMission in class DispatchMission cannot be applied to given types` | `super(...)` missing or arguments in the wrong order | `super(id, distanceKm, loadKg);` first |
| `cannot find symbol ... isCritical()` on the request | `NewAirMission` has `getCritical()` | `request.getCritical()` |
| `incompatible types: possible lossy conversion from long to int` | Returned a `long` where `int` is needed | `Math.toIntExact(energy)` |
| 500 with `UnsupportedOperationException: Task 3` | A TODO is still the starter line | Implement it |
| 500 for an unknown id | Handler method missing, or not throwing `MissionNotFoundException` | §8.1, §8.3 |
| 404 for a whole endpoint you just wrote | Not restarted, or path typo | Restart; compare with §9 |
| 400 on `/plan` | `capacity` missing, or `?capacity=abc` | Send a number |
| `Port 8080 was already in use` | Old run still going | Stop all runs, run again |
| Red `org.springframework` imports | Maven not loaded / wrong JDK | Reload Maven; set JDK 25 |

---

## 13. The eight patterns to memorise

Write these from memory until you can do it in 10 minutes.

```java
// 1. SUBCLASS SHAPE
public class X extends Parent {
    private final int extra;
    public X(long id, int a, int b, int extra) { super(id, a, b); this.extra = extra; }
    public int getExtra() { return extra; }
}

// 2. SAFE FORMULA OVERRIDE
@Override public int energyUnits() {
    long v = (3L * getA()) + (8L * getB()) + (2L * extra);
    return Math.toIntExact(v);
}

// 3. IF-PRIORITY OVERRIDE
@Override public int priority() { if (flag) { return 5; } return 2; }

// 4. FIND OR 404
Parent p = repo.find(id);
if (p == null) { throw new XNotFoundException(id); }

// 5. RANGE VALIDATION + DUPLICATE, THEN SAVE
if (r.getA() < 1 || r.getA() > 10000) { throw new IllegalArgumentException("a must be between 1 and 10000."); }
if (repo.exists(r.getId())) { throw new IllegalArgumentException("Duplicate ID " + r.getId()); }
repo.save(obj);

// 6. FILTER + SORT + GREEDY
List<Parent> list = new ArrayList<>();
for (Parent p : repo.all()) { if (!urgentOnly || p.priority() >= 3) { list.add(p); } }
Collections.sort(list, new Comparator<Parent>() {
    @Override public int compare(Parent a, Parent b) {
        int r = Integer.compare(b.priority(), a.priority()); if (r != 0) { return r; }   // DESC
        r = Integer.compare(a.energyUnits(), b.energyUnits()); if (r != 0) { return r; } // ASC
        return Long.compare(a.getId(), b.getId());                                       // ASC
    }
});
int used = 0;
for (Parent p : list) {
    int e = p.energyUnits();
    if (e <= capacity - used) { selected.add(new Line(p.getId(), p.getClass().getSimpleName(), p.priority(), e)); used += e; }
    else { skipped.add(p.getId()); }
}

// 7. HANDLER
@ExceptionHandler(XNotFoundException.class)
public ResponseEntity<ErrorResponse> nf(XNotFoundException ex) {
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(new ErrorResponse("NOT_FOUND", ex.getMessage()));
}

// 8. ENDPOINTS
@GetMapping("/{id}/energy")
public Map<String, Integer> energy(@PathVariable("id") long id) { return Map.of("energyUnits", service.energy(id)); }
@PostMapping("/air") @ResponseStatus(HttpStatus.CREATED)
public Parent add(@RequestBody NewX request) { return service.add(request); }
@GetMapping("/plan")
public Plan plan(@RequestParam("capacity") int capacity,
                 @RequestParam(name = "urgentOnly", defaultValue = "false") boolean urgentOnly) { return service.plan(capacity, urgentOnly); }
```

---

## 14. The blank-page drill

Do this three times before the exam (timetable §2):

1. Unzip a **fresh** starter into a new folder.
2. Start a 100-minute timer.
3. Write everything without this guide. Restart and test after each task.
4. At 90 minutes: stop, run the six Section B checks, zip.
5. Compare with your lecturer's solution. For every difference, ask: *does mine still meet the paper's rule?* Different code that obeys the rules is fine.
6. Write your mistakes on one card. Read the card before the next attempt.

**Target:** attempt 1 with the guide, attempt 2 under 100 minutes without it, attempt 3 under 75 minutes.

**Next: M2, an advanced paper in the same format, with its solution taught step by step.**
