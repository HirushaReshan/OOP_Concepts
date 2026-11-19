# START HERE: Your 2-Hour Plan for the Dispatch Mock Exam

You have the exam paper (Emergency Dispatch Energy Planner), the starter project, the solution, and your old study files. This pack turns all of that into a short, focused route.

## What the exam really is (in one paragraph)

You get a half-built Spring Boot project. You fill in **TODO** methods, create **one new class** (`AirMission`), and add **three endpoints**. Then the marker **runs your app** and checks 6 behaviours (Section B, 20 marks). Everything in the paper is one story:

> Missions (ground or air) each **cost energy** and have a **priority**. Given a **budget**, pick missions **best-first** while they fit, and report what was picked and what was skipped.

That's it. Four ideas: **inheritance** (two mission types), **polymorphism** (call `energyUnits()` without caring which type), **a greedy loop**, and **HTTP plumbing** (controller → service → repository + error handling).

## READ THIS FIRST: what is different from what you studied

Your old files (S6, Phase 8) were built for *other* papers (Java 17, Boot 3.3.5, **records**, `ApiError`). This paper is different. Do not copy old habits blindly:

| Old habit | This paper |
| :-- | :-- |
| Request/response types as `record` | **Forbidden.** "Must be ordinary Java classes, not Java records." The starter already gives you `NewAirMission`, `PlanLine`, `DispatchPlan`, `ErrorResponse` as normal classes (getters + setters). Use `getX()`, never `x()`. |
| `ApiError` | It's **`ErrorResponse(code, message)`** in the `exception` package |
| A `dto` package | **Do not create one.** Packages: controller, exception, model, repository, service |
| Java 17 / Boot 3 | **Java 25 / Spring Boot 4.1.1**, Swagger UI included |
| Port 8081/8082 | `application.properties` says **8080** and so does the paper. (The starter's README wrongly says 8082. Trust the properties file and the console line `Tomcat started on port 8080`.) |
| Streams are fine | Allowed, but the model solution uses plain `for` loops + `Comparator`. Use whichever you can write without thinking. This pack shows loops. |

## Reading order and time budget (110 min, 10 min spare)

| # | File | What it does | Time |
| :-- | :-- | :-- | :-- |
| 1 | `01_Decode_the_Paper.md` | **How to understand the questions.** Every task translated to plain English, with the traps | 20 min |
| 2 | `02_Concepts_Refresher.md` | Only the Java/Spring you need, with this exam's examples | 25 min |
| 3 | `03_Build_It_Step_by_Step.md` | **Do it with your hands:** rebuild the project from the starter in exam order, with checks | 45 min |
| 4 | `04_Drills_and_Variations.md` | Hand-trace drills + "what if the real exam is a different story?" | 15 min |
| 5 | `05_Cheat_Sheet.md` | One page for tomorrow morning | 5 min |
| - | `section-b-checks.http` | IntelliJ HTTP Client file with every Section B request | use in step 3 |

**Rule for step 3:** don't copy-paste from the solution zip. Open the **starter** zip, and type the answers yourself, looking at my code only when stuck. Typing is what makes it stick.

## The 5 things that win or lose marks

1. **The app must start.** Section B only scores a running app. A half-typed method that doesn't compile = 0 for all 20 Section B marks. If you can't finish a method, leave the starter's `throw new UnsupportedOperationException(...)` so it still compiles.
2. **`long` maths + `Math.toIntExact`.** Both formulas and `addAir` repeat this. It's easy marks you get by writing `2L * ...`.
3. **Greedy keeps going.** When a mission doesn't fit, add it to `skipped` and **continue**. Never `break`.
4. **Filtered-out ≠ skipped.** With `urgentOnly=true`, priority < 3 missions vanish from both lists.
5. **Errors are HTTP codes, not 200s.** Service throws; the handler converts: `IllegalArgumentException` → 400 `BAD_REQUEST`, `MissionNotFoundException` → 404 `NOT_FOUND`.

## If you only have 45 minutes left

Read `05_Cheat_Sheet.md`, then do `03_Build_It_Step_by_Step.md` parts 1-6 straight through and run `section-b-checks.http`.
