# 05: One-Page Cheat Sheet (Dispatch Mock)

## The story in 3 lines
Missions cost **energy**, have **priority**. Sort best-first (**priority ↓, energy ↑, id ↑**). Walk the list: **fits → select, else skip & continue**. `urgentOnly` = pre-filter `priority ≥ 3` (filtered ones in **neither** list).

## Formulas and data
| | Formula | Priority |
| :-- | :-- | :-- |
| Ground `(id, km, kg, medical)` | `2L*km + 5L*kg` | medical 3 / else 1 |
| Air `(id, km, kg, wind, critical)` | `3L*km + 8L*kg + 2L*wind` | critical 5 / else 2 |

Seeds: `101 (12,3,true)` → 39, p3 · `102 (4,2,false)` → 18, p1 · `103 (8,1,3,true)` → 38, p5 · POST `104` → 30, p2

## Expected answers
| Check | Result |
| :-- | :-- |
| `/missions/101/energy` | `{"energyUnits":39}` |
| `/missions/103/energy` | `{"energyUnits":38}` |
| plan 60, false | selected `[103,102]` skipped `[101]` used 56 rem 4 |
| plan 60, true | selected `[103]` skipped `[101]` (102 nowhere) |
| POST 104 | **201** |
| plan 125, false | selected `[103,101,104,102]` used 125 rem 0 |
| capacity ≤ 0 / dup ID / range | **400** `BAD_REQUEST` |
| unknown ID | **404** `NOT_FOUND` |

## Code skeletons (type from memory)

```java
// SAFE ARITHMETIC
long e = (2L * getDistanceKm()) + (5L * getLoadKg());
return Math.toIntExact(e);

// SUBCLASS
public class AirMission extends DispatchMission {
    private final int windPenalty; private final boolean critical;
    public AirMission(long id, int distanceKm, int loadKg, int windPenalty, boolean critical) {
        super(id, distanceKm, loadKg);
        this.windPenalty = windPenalty; this.critical = critical;
    }
    // getters + @Override energyUnits(), priority()
}

// SORT
list.sort(Comparator.comparingInt(DispatchMission::priority).reversed()
        .thenComparingInt(DispatchMission::energyUnits)
        .thenComparingLong(DispatchMission::getId));

// GREEDY
int used = 0;
for (DispatchMission m : list) {
    int e = m.energyUnits();
    if (e <= capacity - used) { selected.add(new PlanLine(m.getId(), m.getClass().getSimpleName(), m.priority(), e)); used += e; }
    else { skipped.add(m.getId()); }
}
return new DispatchPlan(capacity, used, capacity - used, selected, skipped);

// LOOKUP
DispatchMission m = repo.find(id);
if (m == null) throw new MissionNotFoundException(id);
return m.energyUnits();

// ADD: validate -> duplicate -> build -> calculate -> save
if (repo.exists(id)) throw new IllegalArgumentException("...");
AirMission a = new AirMission(...); a.energyUnits(); repo.save(a); return a;

// HANDLER
@ExceptionHandler(IllegalArgumentException.class)
public ResponseEntity<ErrorResponse> bad(IllegalArgumentException ex) {
    return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(new ErrorResponse("BAD_REQUEST", ex.getMessage()));
}
// NOT_FOUND: same with MissionNotFoundException, HttpStatus.NOT_FOUND, "NOT_FOUND"

// CONTROLLER (class already has @RequestMapping("/missions"))
@GetMapping("/{id}/energy")  Map<String,Integer> energy(@PathVariable("id") long id)  → Map.of("energyUnits", service.energy(id))
@PostMapping("/air") @ResponseStatus(HttpStatus.CREATED)  DispatchMission addAir(@RequestBody NewAirMission request)
@GetMapping("/plan")  DispatchPlan plan(@RequestParam("capacity") int capacity,
                                       @RequestParam(name="urgentOnly", defaultValue="false") boolean urgentOnly)
```

## Validation ranges
`id 1-10000` · `distanceKm 1-10000` · `loadKg 0-10000` · `windPenalty 0-10000` · duplicate ID → reject · capacity `> 0`

## Rules that cost marks
- ❌ records · ❌ `dto` package · ❌ `instanceof` · ❌ `break` in the greedy loop · ❌ HTTP 200 for errors
- ❌ saving before validating/calculating · ❌ filtering inside `else` · ❌ logic in the controller
- ❌ `.reversed()` at the end of the comparator chain · ❌ `2 *` instead of `2L *`
- ✅ `getCritical()` on `NewAirMission` (not `isCritical()`)
- ✅ Keep `GET /missions`, the repository API, the parent class, the base-package main class

## Error lookup
| Symptom | Fix |
| :-- | :-- |
| Red `AirMission` everywhere | Create it in `model`; check `package` line |
| `UnsupportedOperationException` at runtime | A starter TODO is still unfinished |
| 404 on a valid URL | Doubled `/missions` in annotation; app not restarted |
| 415 on POST | Add header `Content-Type: application/json` |
| 500 | Read top of stack trace; usually unhandled exception type or null |
| Mission 104 gone | Restart wipes in-memory data; POST again |
| Port confusion | Trust console: `Tomcat started on port(s): 8080` |

## Time plan (100 min)
`0-12` read paper + starter · `12-20` models + seed · `20-40` service · `40-50` handler + controller · `50-70` Section B · `70-85` audit bullets · `85-100` ZIP the **whole project**, open ZIP, submit.
