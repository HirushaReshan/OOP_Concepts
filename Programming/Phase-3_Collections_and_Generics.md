# Phase 3: Collections, Generics and the Service Class

**Module:** 5COSC019W Object Oriented Programming
**Before this:** Phase 2 (private fields, getters, setters, validation, UML).
**Goal:** by the end you can store any number of objects in a `List` or `Map`, search and filter them, handle "not found" safely with `Optional`, and put all of that inside a service class with one clear job.
**Time needed:** about 3 hours.

**Built from your files:** Lecture 3 (Collections, Generics & the Service Layer), Tutorial 3 (From One Product to a Catalogue).

> **Why this phase matters for the exam.** Task 3 (20 marks) is "Services, validation and exceptions". The service in your exam starter will hold its objects in a collection and search them with exactly the code in this guide. Lecture 6's mock briefing also says the GET-all check must return "the seeded items", which is a list.

---

## Where we are on the roadmap

| Phase | Title | Status |
| :-- | :-- | :-- |
| Pre-Phase 0 | Absolute Java Basics | Done |
| Phase 1 | Classes and Objects | Done |
| Phase 2 | Encapsulation and UML Class Diagrams | Done |
| **Phase 3** | **Collections and Generics** | **You are here** |
| Phase 4 | Inheritance | Next |
| Phase 5 | Polymorphism and Abstraction | |
| Phase 6 | Exceptions | |
| Phase 7 | Dependencies and Layered Design | |
| Phase 8 | Exam Consolidation | |

All examples use this `Product` class (the Phase 2 version, with setters added, exactly as Tutorial 3 asks):

```java
public class Product {

    private Long id;
    private String name;
    private double price;

    public Product() {}

    public Product(Long id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public Long getId() { return id; }
    public String getName() { return name; }
    public double getPrice() { return price; }

    public void setId(Long id) { this.id = id; }
    public void setName(String name) { this.name = name; }
    public void setPrice(double price) { this.price = price; }
}
```

---

## 1. Why arrays are not enough

You met arrays in Pre-Phase 0. Lecture 3 starts by showing why they do not suit a shop catalogue.

```java
Product[] products = new Product[500];
```

| Array problem | Why it hurts |
| :-- | :-- |
| You must know the size (500) when you write the line. | A shop does not know how many products it will sell next month. |
| There is no `add()` and no `remove()`. | The length never changes. |
| Growing means building a bigger array and copying everything across by hand. | Tedious and error-prone. |

Lecture 3's key takeaway: **an array is a fixed-length box. A collection is a box that resizes itself.**

A **collection** is an object whose job is to hold a group of other objects, and which grows and shrinks as you add and remove them.

- **Analogy:** an array is an egg carton: exactly 12 slots, forever. A collection is a shopping bag: put in as much as you like, take things out, and it adjusts.

Java's collections live in the package `java.util`, so files that use them start with imports such as:

```java
import java.util.ArrayList;
import java.util.List;
```

---

## 2. `List` and `ArrayList`

### 2.1 Your first list

```java
List<Product> products = new ArrayList<>();
```

| Piece | What it means |
| :-- | :-- |
| `List` | The type of the variable: "an ordered collection that can add, get and remove". |
| `<Product>` | What goes inside: only `Product` objects. The angle brackets are explained in Section 3. |
| `products` | The variable's name. Plural, because it holds many. |
| `=` | "Becomes." |
| `new ArrayList<>()` | Build a new, **empty** `ArrayList`. |
| `<>` | Called the **diamond**. It is empty because Java works out the type (`Product`) from the left side. |

### 2.2 Why `List` on the left and `ArrayList` on the right?

Lecture 3: *"One is a contract, one is a class."*

| | `List` | `ArrayList` |
| :-- | :-- | :-- |
| What it is | An **interface**: a promise of what something can do (add, get, remove), with no code of its own | A real **class** that keeps that promise, using a resizing array inside |
| Can you write `new` on it? | **No.** `new List<>()` is a compile error. | Yes |

An **interface** is a list of abilities a class promises to provide, without saying how. Phase 5 teaches interfaces properly. For now:

- **Analogy:** "driving licence" is the contract: whoever holds one can drive. `ArrayList` is a particular driver who holds one. Your code only needs "someone who can drive", so it asks for a `List`. Later you can swap in a different driver (such as `LinkedList`, another class that keeps the same contract) and no other line changes.

Lecture 3, quick check Q1: *"Why do we write `List` on the left and `ArrayList` on the right?"* → **The variable depends only on the contract. Swap in a different implementation later and no other line changes.** This idea is called **programming to the interface**, and it returns in Phases 5 and 7.

### 2.3 The six operations you will actually use

```java
List<Product> products = new ArrayList<>();

products.add(new Product(1L, "Laptop", 899.99));
products.add(new Product(2L, "Mouse", 24.99));

System.out.println(products.size());              // 2
System.out.println(products.get(0).getName());    // Laptop
System.out.println(products.isEmpty());           // false

products.remove(1);                               // the Mouse is gone
System.out.println(products.size());              // 1
```

| Method | What it does | Notes |
| :-- | :-- | :-- |
| `add(item)` | Adds to the end. | The list grows by one. |
| `get(index)` | Returns the item at that position. | **Starts at 0**, like arrays. |
| `size()` | How many items right now. | A method with brackets. (Arrays use `.length` with no brackets; lists use `.size()`.) |
| `remove(index)` or `remove(item)` | Removes it; everything after shifts left to fill the gap. | |
| `contains(item)` | `true` if the list holds an equal item. | Uses `.equals()`, not `==`. |
| `isEmpty()` | `true` if there are no items. | Clearer than `size() == 0`. |

`products.get(0).getName()` is method chaining (Phase 2): `get(0)` returns a `Product`, then `.getName()` is called on it.

Lecture 3, Q2: *"A list has 3 elements. What does `get(3)` return?"* → **Nothing. It throws `IndexOutOfBoundsException`.** Valid positions are 0, 1 and 2.

### 2.4 Walking through a list

**The for-each loop** (Pre-Phase 0, Section 12.4) works on lists too. This is the one to use:

```java
for (Product p : products) {
    System.out.println(p.getName());
}
```

Read it as "for each `Product p` in `products`". On each pass, `p` refers to the next product.

**`forEach` with a lambda** does the same in one line:

```java
products.forEach(p -> System.out.println(p.getName()));
```

`p -> ...` is a **lambda**, explained in Section 6.

**Finding one item the long way:**

```java
public Product findById(Long id) {
    for (Product p : products) {
        if (p.getId().equals(id)) {
            return p;
        }
    }
    return null;
}
```

| Line | What it means |
| :-- | :-- |
| `for (Product p : products)` | Check each product in turn. |
| `if (p.getId().equals(id))` | Is this the one? (Why `.equals` and not `==`: Section 4.3.) |
| `return p;` | Found it. `return` leaves the method immediately, so the loop stops. |
| `return null;` | Only reached if the loop finished without finding a match. Section 7 replaces this with something safer. |

---

## 3. Generics: the `<T>` in the box

### 3.1 What the angle brackets do

**Generics** are the angle brackets that say what type a collection holds. `List<Product>` means "a list of `Product`s, and nothing else".

- **Analogy (Lecture 3):** a `List<Product>` is a **labelled box**. Only Products go in, so you always know what comes out, and you never have to unwrap and check.

Without generics, a list accepts anything, and mistakes only appear while the program runs:

```java
// WITHOUT generics: a "raw" list. Anything goes.
List products = new ArrayList();
products.add(new Product(1L, "Laptop", 899.99));
products.add("oops");                        // compiles!
Product p = (Product) products.get(1);       // ClassCastException at runtime
```

```java
// WITH generics: one kind only.
List<Product> products = new ArrayList<>();
products.add(new Product(1L, "Laptop", 899.99));
products.add("oops");                        // will not compile
Product p = products.get(0);                 // no cast needed
```

`(Product)` in the first version is a **cast**: forcing Java to treat a value as a particular type. If the value is not really that type, the program crashes with `ClassCastException`. Generics make the cast unnecessary.

### 3.2 Will it compile? (Lecture 3 "Ask the Room")

```java
List<Product> items = new ArrayList<>();

items.add(new Product(1L, "Mug", 8.50));   // A
items.add("Mug");                          // B
String s = items.get(0);                   // C
int total = items.size();                  // D
```

| Line | Compiles? | Why |
| :-- | :-- | :-- |
| A | Yes | A `Product` is exactly what the box is labelled for. |
| B | **No** | A `String` is not a `Product`. |
| C | **No** | `get(0)` hands back a `Product`, which does not fit in a `String` variable. |
| D | Yes | `size()` returns an `int`, whatever the list holds. |

Three of these four lines are rejected **before the program runs**. That is the benefit.

### 3.3 What generics buy you (Lecture 3)

| Benefit | Meaning |
| :-- | :-- |
| Type safety | The wrong type is a red underline in the IDE, not a crash in production. |
| No casting | `get()` already returns the right type. |
| Readable intent | `List<Product>` tells the next developer what is inside. |
| Autocomplete | IntelliJ offers `getName()` on the loop variable because it knows the type. |

Report sentence from Lecture 3: *"The catalogue is held in a `List<Product>`, so the compiler guarantees every element is a `Product`."*

The letter **`T`** in `List<T>` is just a placeholder meaning "some type, to be chosen later". `List<Product>` fills it in.

---

## 4. Wrapper types and the `.equals()` rule

### 4.1 Collections cannot hold primitives

```java
List<int> numbers = new ArrayList<>();       // COMPILE ERROR
List<Integer> numbers = new ArrayList<>();   // correct
```

Collections can only hold **objects**, and primitives (`int`, `long`, `double`, `boolean`) are not objects. Each primitive has a **wrapper class**: an object version of the same value. You met `Long` in Phase 1.

| Primitive | Wrapper |
| :-- | :-- |
| `int` | `Integer` |
| `long` | `Long` |
| `double` | `Double` |
| `boolean` | `Boolean` |
| `char` | `Character` |

Java converts between them automatically when you add and get. This is called **autoboxing** (primitive → wrapper) and **unboxing** (wrapper → primitive).

```java
List<Integer> marks = new ArrayList<>();
marks.add(65);            // 65 is boxed into an Integer automatically
int first = marks.get(0); // unboxed back to int
```

### 4.2 Why `Long id` instead of `long id`?

`Product` uses `Long id` (capital L) because a wrapper can be **`null`**. A brand-new product arriving from a web form has **no id yet**, and `null` says that honestly. A primitive `long` cannot be `null`; it would have to be `0`, which might look like a real id.

### 4.3 The `==` trap with wrappers

Tutorial 3: *"`.equals()` compares values. Always use it for `Long` and `String`. `==` compares references and will betray you."*

`Long` is a class, so a `Long` variable holds a reference. `==` asks "same object?" (Phase 1, Section 10.4). Java happens to reuse objects for small numbers (−128 to 127), so `==` **appears** to work in small tests and then fails with bigger ids:

```java
Long a = 127L;
Long b = 127L;
System.out.println(a == b);        // true  (by luck)

Long c = 1000L;
Long d = 1000L;
System.out.println(c == d);        // false (!)
System.out.println(c.equals(d));   // true
```

> **Rule.** Compare `Long`, `Integer`, `String` and every other object with **`.equals()`**. Use `==` only for primitives (`int`, `long`, `double`, `boolean`) and for checking `null`.

---

## 5. `Map`: finding by key in one step

### 5.1 The problem with searching a list

To find product 499 in a list of 500, the loop in Section 2.4 may check all 500. Double the catalogue and you double the work. This is called **linear search**, and Lecture 3 notes it is written **O(n)**: the work grows in step with the number of items, `n`.

### 5.2 What a map is

A **map** stores **key → value pairs**. You look up a value by its key and go straight to it, however many entries there are.

- **Analogy: a dictionary.** You do not read every page to find "penguin". You go straight to P. The word is the key; the definition is the value.
- **Analogy: a coat check.** You hand over a coat and get ticket 42. Later, ticket 42 gets you exactly your coat, without searching every hook.

```java
Map<Long, Product> catalogue = new HashMap<>();

catalogue.put(1L, new Product(1L, "Laptop", 899.99));
catalogue.put(2L, new Product(2L, "Mouse", 24.99));

System.out.println(catalogue.get(2L).getName());   // Mouse
System.out.println(catalogue.containsKey(7L));     // false
catalogue.remove(1L);                              // the Laptop is gone
System.out.println(catalogue.size());              // 1

List<Product> all = new ArrayList<>(catalogue.values());
```

| Piece | What it means |
| :-- | :-- |
| `Map<Long, Product>` | Two types now: the **key type** (`Long`), then the **value type** (`Product`). |
| `new HashMap<>()` | `Map` is the contract (interface); `HashMap` is the usual class that keeps it. Same idea as `List`/`ArrayList`. |
| `put(key, value)` | Add a pair. **If the key already exists, the old value is replaced.** There is no `add()` on a map. |
| `get(key)` | The value for that key, in one step. **Returns `null` if the key is missing.** |
| `containsKey(key)` | `true` if the key is present. |
| `remove(key)` | Delete that pair. |
| `values()` | All the values, as a collection. Wrapping it in `new ArrayList<>(...)` gives you a `List`. |
| `keySet()` | All the keys. |

Keys are **unique**. Values do not have to be.

**`HashMap` does not keep any order.** If you print it, entries may come out in any order. Never rely on it.

Walking through a map:

```java
for (Product p : catalogue.values()) {
    System.out.println(p.getName());
}
```

### 5.3 List or Map? (Lecture 3)

| Use a `List` when... | Use a `Map` when... |
| :-- | :-- |
| You need every item, in order. | You look things up by a unique key. |
| Order matters, or duplicates are allowed. | Each key appears once; putting the same key again replaces the value. |
| Finding one item means scanning the list. | `get(key)` goes straight there. |

Lecture 3's vote:

| Data | Choice | Why |
| :-- | :-- | :-- |
| All products, in order added | List | The endpoint returns the whole catalogue in order. |
| Look up a user by email | Map | The email is a unique key. |
| Items in a shopping basket | List | Order matters, and the same item may appear twice. |
| Country code → country name | Map | `"GB"` → `"United Kingdom"`: a textbook key-value pair. |
| Comments under a blog post | List | In time order; duplicates are fine. |
| Product id → Product object | Map | Exactly what the Phase 7 repository holds. |

The test question for every map: **"What is the key, and can two entries ever share it?"** If two could share it, it cannot be a map key.

---

## 6. Lambdas and streams

### 6.1 Lambdas: a small function passed as a value

A **lambda** is a tiny unnamed method that you write inline and hand to another method. The arrow `->` separates the input from what to do with it.

```java
p -> p.getPrice() < 50
```

Read it as: "**given** a product `p`, **give back** whether its price is below 50".

| Piece | What it means |
| :-- | :-- |
| `p` | The input. You choose the name; `p` for product is the habit. |
| `->` | "Goes to" / "gives back". |
| `p.getPrice() < 50` | The result: here, `true` or `false`. |

- **Analogy:** handing a sorting assistant an instruction card that says "keep anything cheaper than 50". The card is the lambda; the assistant (the stream) applies it to each item.

Lecture 3 says to use these "as a recipe this week". The patterns below cover what you need.

### 6.2 Streams: an assembly line

A **stream** passes the items of a collection, one at a time, through a series of steps, like a factory conveyor belt with stations.

**Filter: keep only matching items** (Lecture 3 and Tutorial 3):

```java
public List<Product> findByName(String name) {
    return products.stream()
            .filter(p -> p.getName().contains(name))
            .toList();
}
```

| Step | Read it as |
| :-- | :-- |
| `products.stream()` | "Put every product on the conveyor belt, one at a time." |
| `.filter(p -> p.getName().contains(name))` | "Keep only those whose name contains the search text." (`contains` is a `String` method: does this text appear inside?) |
| `.toList()` | "Collect the survivors into a new list." |

The same thing the long way:

```java
List<Product> found = new ArrayList<>();
for (Product p : products) {
    if (p.getName().contains(name)) {
        found.add(p);
    }
}
return found;
```

Both are correct. The stream version is shorter and is what your lecture code uses.

> `.toList()` needs Java 16 or newer. If IntelliJ cannot find it, use `.collect(Collectors.toList())` and add `import java.util.stream.Collectors;` (Tutorial 3's tip).

`contains` is case-sensitive: searching "lap" will not find "Laptop". To ignore case, compare lower-case versions: `p.getName().toLowerCase().contains(name.toLowerCase())`.

**Find the first match** (used for find-by-id):

```java
public Optional<Product> getProductById(Long id) {
    return products.stream()
            .filter(p -> p.getId().equals(id))
            .findFirst();
}
```

`.findFirst()` stops at the first item that survives the filter and returns it wrapped in an **`Optional`** (Section 7).

### 6.3 Other useful stream steps

Not on your slides, but common in exam-style service methods:

```java
long cheapCount = products.stream()
        .filter(p -> p.getPrice() < 50)
        .count();                                   // how many match

boolean anyExpensive = products.stream()
        .anyMatch(p -> p.getPrice() > 500);         // does at least one match?

List<String> names = products.stream()
        .map(p -> p.getName())
        .toList();                                  // turn each Product into its name

List<Product> byPrice = products.stream()
        .sorted(Comparator.comparing(Product::getPrice))
        .toList();                                  // sorted, cheapest first
```

| Step | What it does |
| :-- | :-- |
| `.count()` | How many items reached the end. Returns a `long`. |
| `.anyMatch(...)` | `true` if at least one item matches. |
| `.map(...)` | Transforms each item into something else. |
| `.sorted(Comparator.comparing(Product::getPrice))` | Sorts by price. `Product::getPrice` is shorthand for `p -> p.getPrice()`, called a **method reference**. `Comparator` needs `import java.util.Comparator;`. |

### 6.4 Removing safely: `removeIf`

```java
public void deleteProduct(Long id) {
    products.removeIf(p -> p.getId().equals(id));
}
```

`removeIf` deletes every item for which the lambda is `true`.

Why not remove inside a for-each loop? Because changing a list while a for-each loop is walking through it crashes with **`ConcurrentModificationException`**. ("Concurrent modification" = "it was changed while being read".)

- **Analogy:** someone pulling chairs out of a row while you are counting the chairs. You lose your place.

---

## 7. `Optional`: "a product, or nothing"

### 7.1 The problem with returning `null`

The long-way `findById` (Section 2.4) returns `null` when nothing matches. The caller may forget to check, and later calls a method on it: `NullPointerException` (Phase 1). Tutorial 3's warm-up shows this exact crash with a map:

```java
Product q = byId.get(99L);            // compiles; q is null
System.out.println(q.getName());      // compiles; crashes at runtime
```

### 7.2 What `Optional` is

**`Optional<Product>`** is a small wrapper object that **either contains one `Product` or is empty**. Tutorial 3: *"A Product, or nothing. Forces the caller to handle the missing case instead of quietly returning null."*

- **Analogy:** a parcel that might be empty. The method hands you the box. You must open it and check before you use what is inside. With `null`, you were handed nothing at all, and only found out when you tried to use it.

The return type `Optional<Product>` itself is a warning sign to the caller: "this might not find anything".

### 7.3 Opening the box

```java
Optional<Product> result = service.getProductById(2L);

if (result.isPresent()) {
    System.out.println(result.get().getName());
} else {
    System.out.println("Not found");
}
```

| Method | What it does |
| :-- | :-- |
| `isPresent()` | `true` if something is inside. |
| `isEmpty()` | `true` if nothing is inside. |
| `get()` | Takes the value out. **Only call it after checking.** On an empty `Optional` it throws `NoSuchElementException`. |
| `orElse(other)` | The value if present, otherwise `other`. `orElse(null)` gets you back to the old null problem. |
| `orElseThrow()` | The value if present, otherwise throw an exception. Phase 6 uses this to produce a proper "404 Not Found". |

Lecture 3's controller uses `.orElse(null)` and labels it "a placeholder we replace with a real 404 in Lecture 6". Remember that: `orElse(null)` is the **unfinished** version.

Making your own `Optional` values:

```java
Optional<Product> some = Optional.of(laptop);     // a box with something
Optional<Product> none = Optional.empty();        // an empty box
```

---

## 8. The service class: one class, one job

### 8.1 The problem: everything in one class

Lecture 3 shows a Spring controller doing three jobs: answering web requests, inventing ids, and storing products. The same problem appears in plain Java whenever `main` holds the list and every rule.

| Problem | Effect |
| :-- | :-- |
| Three jobs in one class | Hard to read, hard to change. |
| Untestable on its own | To check the id rule you must run the whole web application. |
| Nothing can reuse it | A second class cannot reach that list. |

### 8.2 The fix: a service class

A **service** is a class that **owns a collection and the business rules about it**, and knows nothing about how requests arrive (web, keyboard, file). **Business rules** (also called **business logic**) are the real-world rules of the problem: "ids start at 1 and go up", "price cannot be negative".

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

public class ProductService {

    private List<Product> products = new ArrayList<>();
    private Long nextId = 1L;

    public List<Product> getAllProducts() {
        return products;
    }

    public Optional<Product> getProductById(Long id) {
        return products.stream()
                .filter(p -> p.getId().equals(id))
                .findFirst();
    }

    public Product addProduct(Product product) {
        product.setId(nextId++);
        products.add(product);
        return product;
    }

    public void deleteProduct(Long id) {
        products.removeIf(p -> p.getId().equals(id));
    }

    public List<Product> findByName(String name) {
        return products.stream()
                .filter(p -> p.getName().contains(name))
                .toList();
    }
}
```

| Line | What it means |
| :-- | :-- |
| `private List<Product> products = new ArrayList<>();` | The catalogue. **Private**: nobody outside can reach in and change it. A field can be given its starting value right where it is declared. |
| `private Long nextId = 1L;` | Remembers which id to hand out next. |
| `getAllProducts()` | Returns the whole list. |
| `getProductById(Long id)` | Returns an `Optional`: found, or empty. |
| `addProduct(Product product)` | Gives the product the next id, stores it, and returns it. |
| `product.setId(nextId++);` | **Post-increment**: use the current value of `nextId`, **then** add 1. First product gets 1, and `nextId` becomes 2. (`++nextId` would add first, so the first product would get 2.) |
| `deleteProduct(Long id)` | Removes any product with that id. |
| `findByName(String name)` | Filters by name. |

Tutorial 3's key point: **there is not one web word in this class.** It would work unchanged in a desktop app.

Using it from `main`:

```java
public class Main {
    public static void main(String[] args) {
        ProductService service = new ProductService();

        service.addProduct(new Product(null, "Laptop", 899.99));
        service.addProduct(new Product(null, "Mouse", 24.99));
        service.addProduct(new Product(null, "Monitor", 249.99));

        for (Product p : service.getAllProducts()) {
            System.out.println(p.getId() + " " + p.getName());
        }

        System.out.println(service.getProductById(2L).isPresent());   // true
        System.out.println(service.getProductById(99L).isPresent());  // false

        service.deleteProduct(2L);
        System.out.println(service.getAllProducts().size());          // 2

        System.out.println(service.findByName("Mon").size());         // 1
    }
}
```

**Output:**

```
1 Laptop
2 Mouse
3 Monitor
true
false
2
1
```

Products are created with `null` as the id because the **service** decides ids. The caller never invents one.

### 8.3 How Spring uses this (the Spring chat covers the code)

In your Spring project, the same class gets `@Service` written above it. That label tells Spring to create one `ProductService` when the application starts. The controller then receives it (Lecture 3 uses `@Autowired`; Phase 7 explains the better way). Each controller method becomes one line: take the request, call the service, return the answer.

```
 Web request ──> Controller ──> Service ──> List<Product>
 Web response <── Controller <── Service <──
```

Lecture 3's rule: **requests travel down, answers travel back up. The service never calls the controller.**

Where does each line of code belong? Lecture 3's activity:

| Code | Belongs in | Why |
| :-- | :-- | :-- |
| Reject a product whose price is below zero | **Service** | A business rule. |
| Read the id out of the URL | Controller | Pure web handling. |
| Give a new product the next id | **Service** | The service owns the ids. |
| Return "404 Not Found" when nothing matches | Controller | 404 is a web status code. |
| Filter the catalogue to products under £50 | **Service** | Filtering is logic. |

Lecture 3's test: **if you deleted every annotation, would the class still make sense?** A service: yes. A controller: no.

---

## 9. Cohesion and coupling

Two words your coursework report is marked on (LO3).

| Word | Meaning | Good is... | Test |
| :-- | :-- | :-- | :-- |
| **Cohesion** | How focused a class is on one job. | **High**: everything in the class serves the same purpose. | Can you describe the class in one sentence with no "and"? |
| **Coupling** | How much classes depend on each other. | **Low**: classes lean on each other as little as possible, in one direction. | If you change one class, how many other files must you open? |

- **Analogy for cohesion:** a kitchen drawer that holds only cutlery is cohesive. A "junk drawer" with batteries, string and spoons is not.
- **Analogy for coupling:** a lamp with a plug (low coupling: unplug it and move it) versus a lamp wired into the wall (high coupling: moving it means rewiring).

Lecture 3's quick checks:

| Question | Answer |
| :-- | :-- |
| Your `ProductService` also sends confirmation emails. Which principle is broken? | **Cohesion.** Two jobs in one class. Emails belong in their own service. |
| Why should `ProductService` never hold a reference to `ProductController`? | It would make the dependency **circular**. Requests go one way only. |
| Where does the `ArrayList` live after the refactor, and why? | Private, inside `ProductService`. The controller does not know or care how products are stored: encapsulation at the class level. |

Report sentence (Lecture 3): *"The controller depends on the service, but not the reverse: a one-way dependency that keeps coupling low."*

---

## 10. UML: showing collections and dependencies

### 10.1 A collection as an attribute

Show the element type, angle brackets included:

```
- products : List<Product>
+ getProductById(id : Long) : Optional<Product>
```

### 10.2 Two kinds of arrow

| Arrow | Drawn as | Means | Example |
| :-- | :-- | :-- | :-- |
| **Association** | Solid line, open arrowhead `───>` | "holds / has a" (a field of that type, kept long-term) | `ProductService ───> Product` (it holds Products) |
| **Dependency** | **Dashed** line, open arrowhead `- - ->` | "uses" | `ProductController - - -> ProductService` |

Tutorial 3's diagram:

```
+------------------------------------+
|         ProductController          |
+------------------------------------+
| - productService : ProductService  |
+------------------------------------+
| + getAllProducts() : List<Product> |
| + getProductById(id : Long)        |
|        : Product                   |
+------------------------------------+
                 :
                 :  (dashed: depends on / uses)
                 v
+-------------------------------------------------+
|                 ProductService                  |
+-------------------------------------------------+
| - products : List<Product>                      |
| - nextId : Long                                 |
+-------------------------------------------------+
| + getAllProducts() : List<Product>              |
| + getProductById(id : Long) : Optional<Product> |
| + addProduct(product : Product) : Product       |
| + deleteProduct(id : Long) : void               |
| + findByName(name : String) : List<Product>     |
+-------------------------------------------------+
                 |
                 |  (solid: holds)
                 v
+------------------------------+
|           Product            |
+------------------------------+
| - id : Long                  |
| - name : String              |
| - price : double             |
+------------------------------+
| + getId() : Long  ...        |
+------------------------------+
```

Tutorial 3: *"The direction of these arrows is the whole point. Every arrow leaves the controller; none arrives at it."* If you draw an arrow from the service back to the controller, delete it.

---

## 11. Common mistakes

| Mistake | What happens | Fix |
| :-- | :-- | :-- |
| `List<int>` | Compile error. | `List<Integer>` |
| `new List<>()` | Compile error: `List` is an interface. | `new ArrayList<>()` |
| IntelliJ imports `java.awt.List` | Strange errors on `add()`. | Delete it; use `java.util.List`. |
| `get(size())` | `IndexOutOfBoundsException`. | Last index is `size() - 1`. |
| `p.getId() == id` with `Long` | Works for small ids, fails for big ones. | `p.getId().equals(id)` |
| Removing inside a for-each loop | `ConcurrentModificationException`. | `removeIf(...)` |
| Calling `.get()` on an empty `Optional` | `NoSuchElementException`. | Check `isPresent()`, or use `orElseThrow()` (Phase 6). |
| Assuming `HashMap` keeps order | Output in unexpected order. | Use a `List` if order matters. |
| A product in the list has a `null` id | `NullPointerException` inside the filter. | Every product goes through `addProduct`. |
| `nextId++` vs `++nextId` confusion | Ids start at 2. | `setId(nextId++)` |
| Web code inside the service | Low cohesion; hard to test. | Service has no web words. |

---

## 12. Practice

### 12.1 Tutorial 3 warm-up: predict the output

Setup:

```java
List<Product> products = new ArrayList<>();
products.add(new Product(1L, "Laptop", 899.99));
products.add(new Product(2L, "Mouse", 24.99));

Product[] fixed = new Product[2];

Map<Long, Product> byId = new HashMap<>();
byId.put(1L, new Product(1L, "Laptop", 899.99));
```

For each line: does it compile, and what happens?

| # | Statement |
| :-- | :-- |
| 1 | `products.add(new Product(3L, "Mug", 8.50));` |
| 2 | `products.add("Mug");` |
| 3 | `String s = products.get(0);` |
| 4 | `Product p = products.get(5);` |
| 5 | `int n = products.size();` |
| 6 | `fixed.add(new Product(3L, "Mug", 8.50));` |
| 7 | `Product q = byId.get(99L);` |
| 8 | `System.out.println(q.getName());` |

**Answers**

| # | Compiles? | What happens |
| :-- | :-- | :-- |
| 1 | Yes | Added. The list now holds 3. |
| 2 | No | A `String` is not a `Product`. |
| 3 | No | `get(0)` returns a `Product`, which does not fit in a `String`. |
| 4 | Yes | **Runtime:** `IndexOutOfBoundsException`. There is no position 5. |
| 5 | Yes | `n` becomes 2 (or 3 if row 1 ran first). |
| 6 | No | Arrays have no `add()` method. |
| 7 | Yes | `q` becomes `null`: key 99 is not in the map. |
| 8 | Yes | **Runtime:** `NullPointerException`, because `q` is `null`. |

Rows 4 and 8 are the important ones. **The compiler checks types; it cannot know how many items a list holds or which keys a map contains.** Those mistakes only appear at runtime.

### 12.2 Quick questions

| # | Question | Answer |
| :-- | :-- | :-- |
| 1 | Difference between an array and an `ArrayList`? | An array has a fixed length set at creation; an `ArrayList` grows and shrinks. |
| 2 | Why `List` on the left, `ArrayList` on the right? | The variable depends only on the contract, so the implementation can be swapped without changing other lines. |
| 3 | Can a `List<Product>` hold a `String`? | No. The compiler stops you. |
| 4 | What does `map.get(key)` return for a missing key? | `null`. |
| 5 | Why return `Optional<Product>` rather than `Product`? | It makes "not found" explicit and forces the caller to handle it. |
| 6 | In one sentence: what decides whether code belongs in the controller or the service? (Tutorial 3 exit ticket) | Whether it is about the web request (controller) or about the business rules of the data (service). |
| 7 | What do high cohesion and low coupling mean? | Each class does one job; classes depend on each other as little as possible, in one direction. |

### 12.3 Build it: `LibraryService`

Using this `Book` class:

```java
public class Book {

    private Long id;
    private String title;
    private String author;
    private boolean onLoan;

    public Book(Long id, String title, String author) {
        this.id = id;
        this.title = title;
        this.author = author;
        this.onLoan = false;
    }

    public Long getId() { return id; }
    public String getTitle() { return title; }
    public String getAuthor() { return author; }
    public boolean isOnLoan() { return onLoan; }

    public void setId(Long id) { this.id = id; }
    public void setOnLoan(boolean onLoan) { this.onLoan = onLoan; }
}
```

Write `LibraryService` with:

1. a private `List<Book>` and a private `nextId`;
2. `addBook(Book book)` that assigns the next id and returns the book;
3. `findById(Long id)` returning `Optional<Book>`;
4. `findByAuthor(String author)` returning a list (exact match, ignoring case);
5. `getAvailableBooks()` returning books not on loan;
6. `countOnLoan()` returning how many are on loan;
7. `removeBook(Long id)`.

Try for 25 minutes before reading on.

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

public class LibraryService {

    private List<Book> books = new ArrayList<>();
    private Long nextId = 1L;

    public Book addBook(Book book) {
        book.setId(nextId++);
        books.add(book);
        return book;
    }

    public Optional<Book> findById(Long id) {
        return books.stream()
                .filter(b -> b.getId().equals(id))
                .findFirst();
    }

    public List<Book> findByAuthor(String author) {
        return books.stream()
                .filter(b -> b.getAuthor().equalsIgnoreCase(author))
                .toList();
    }

    public List<Book> getAvailableBooks() {
        return books.stream()
                .filter(b -> !b.isOnLoan())
                .toList();
    }

    public long countOnLoan() {
        return books.stream()
                .filter(b -> b.isOnLoan())
                .count();
    }

    public void removeBook(Long id) {
        books.removeIf(b -> b.getId().equals(id));
    }
}
```

```java
LibraryService library = new LibraryService();
library.addBook(new Book(null, "Dune", "Frank Herbert"));
Book b2 = library.addBook(new Book(null, "Emma", "Jane Austen"));
library.addBook(new Book(null, "Persuasion", "Jane Austen"));

b2.setOnLoan(true);

System.out.println(library.findByAuthor("jane austen").size());   // 2
System.out.println(library.getAvailableBooks().size());           // 2
System.out.println(library.countOnLoan());                        // 1
System.out.println(library.findById(3L).get().getTitle());        // Persuasion
System.out.println(library.findById(9L).isPresent());             // false
```

`equalsIgnoreCase` is a `String` method: like `equals`, but treats capitals and small letters as the same.

### 12.4 Stretch (Tutorial 3): swap the list for a map

Rewrite `ProductService` to store a `Map<Long, Product>` instead of a `List<Product>`. **The rule: no method signature may change**, so no code that uses the service has to change.

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;

public class ProductServiceMap {

    private Map<Long, Product> products = new HashMap<>();
    private Long nextId = 1L;

    public List<Product> getAllProducts() {
        return new ArrayList<>(products.values());
    }

    public Optional<Product> getProductById(Long id) {
        return Optional.ofNullable(products.get(id));
    }

    public Product addProduct(Product product) {
        product.setId(nextId++);
        products.put(product.getId(), product);
        return product;
    }

    public void deleteProduct(Long id) {
        products.remove(id);
    }

    public List<Product> findByName(String name) {
        return products.values().stream()
                .filter(p -> p.getName().contains(name))
                .toList();
    }
}
```

`Optional.ofNullable(x)` makes an `Optional` that is empty if `x` is `null` and full otherwise. It is the bridge between a map's `null` and `Optional`.

**Tutorial 3's question: why did the code that uses the service not need to change?** Because the storage is **private** and the methods kept the same signatures. Outside code only ever saw the methods, never the list. That is encapsulation, and it is the same argument Phase 7 makes for the repository.

---

## 13. Glossary

| Term | Plain-English meaning |
| :-- | :-- |
| **`ArrayList`** | The usual class that implements `List`, using a resizing array. |
| **Autoboxing / unboxing** | Automatic conversion between a primitive and its wrapper. |
| **Business logic / rules** | The real-world rules of the problem, kept in the service. |
| **Cast** | Forcing a value to be treated as a particular type: `(Product) x`. |
| **`ClassCastException`** | Runtime error when a cast is wrong. |
| **Cohesion** | How focused a class is on one job. High is good. |
| **Collection** | An object that holds a group of objects and resizes itself. |
| **`ConcurrentModificationException`** | Runtime error from changing a list while a for-each loop is walking it. |
| **Coupling** | How much classes depend on each other. Low is good. |
| **Dependency (UML)** | Dashed arrow: one class uses another. |
| **Diamond `<>`** | Empty angle brackets where Java infers the type. |
| **Generics** | Angle brackets saying what type a collection holds. |
| **`HashMap`** | The usual class that implements `Map`. Keeps no order. |
| **`IndexOutOfBoundsException`** | Runtime error from a list position that does not exist. |
| **Interface** | A contract listing abilities, without code of its own. |
| **Key / value** | In a map: the label you look up by, and what it leads to. |
| **Lambda** | A tiny unnamed method written inline: `p -> p.getPrice() < 50`. |
| **Linear search / O(n)** | Checking items one by one; the work grows with the number of items. |
| **`List`** | An ordered collection contract: add, get, remove, size. |
| **`Map`** | A key → value collection contract. Keys are unique. |
| **Method reference** | Shorthand like `Product::getPrice` for `p -> p.getPrice()`. |
| **`NoSuchElementException`** | Runtime error from calling `get()` on an empty `Optional`. |
| **`Optional`** | A box holding one value or nothing. |
| **Post-increment `x++`** | Use the value, then add 1. |
| **Programming to the interface** | Declaring variables with the contract type (`List`) rather than the class (`ArrayList`). |
| **`removeIf`** | Removes every item matching a lambda, safely. |
| **Service** | A class that owns a collection and the business rules about it, with no web code. |
| **Stream** | A conveyor belt passing a collection's items through steps like `filter`, `map`, `count`. |
| **Wrapper class** | An object version of a primitive: `Integer`, `Long`, `Double`, `Boolean`. |

---

## 14. One-page cheat sheet

```java
import java.util.*;                                  // List, ArrayList, Map, HashMap, Optional

List<Product> list = new ArrayList<>();              // contract on left, class on right
list.add(p);  list.get(0);  list.size();
list.remove(0);  list.isEmpty();  list.contains(p);
for (Product x : list) { }                           // for each x in list

Map<Long, Product> map = new HashMap<>();            // key type, value type
map.put(1L, p);  map.get(1L);  map.containsKey(1L);  // get() gives null if missing
map.remove(1L);  map.values();  map.keySet();

// streams
list.stream().filter(x -> x.getPrice() < 50).toList();        // keep matches
list.stream().filter(x -> x.getId().equals(id)).findFirst();  // Optional<Product>
list.stream().filter(...).count();                            // long
list.stream().anyMatch(x -> x.getPrice() > 500);              // boolean
list.stream().map(x -> x.getName()).toList();                 // List<String>
list.removeIf(x -> x.getId().equals(id));                     // safe delete

// Optional
Optional<Product> r = service.getProductById(id);
if (r.isPresent()) { r.get(); }
r.orElse(null);          // unfinished version
r.orElseThrow();         // Phase 6: becomes a proper "not found"
Optional.ofNullable(map.get(id));

// service pattern
public class XService {
    private List<X> items = new ArrayList<>();
    private Long nextId = 1L;
    public X add(X x) { x.setId(nextId++); items.add(x); return x; }
}
```

| Remember | |
| :-- | :-- |
| Collections hold objects only | `List<Integer>`, not `List<int>` |
| Compare objects with `.equals()` | `==` on `Long` fails above 127 |
| `get(i)` valid from 0 to `size() - 1` | Otherwise `IndexOutOfBoundsException` |
| `map.get()` returns `null` if missing | Wrap with `Optional.ofNullable` |
| Never remove inside a for-each | Use `removeIf` |
| Service: owns the data and rules, no web words | Controller: web only, one line per method |
| High cohesion, low coupling | One job each; arrows point one way |
| UML: solid arrow = holds, dashed arrow = uses | Nothing points back at the controller |

---

## 15. Am I ready for Phase 4?

- [ ] I can explain why a collection is better than an array for a catalogue. (§1)
- [ ] I can create a `List`, and use `add`, `get`, `size`, `remove`, `contains`, `isEmpty`. (§2)
- [ ] I can explain `List` versus `ArrayList`. (§2.2)
- [ ] I can say which of four lines compile, using generics. (§3)
- [ ] I can explain why `Long` ids are compared with `.equals()`. (§4)
- [ ] I can use a `Map` and decide between a `List` and a `Map`. (§5)
- [ ] I can write a stream that filters, and one that finds by id. (§6)
- [ ] I can return and use an `Optional`. (§7)
- [ ] I can write a service class with a private list, an id counter and find/add/delete methods. (§8)
- [ ] I can define cohesion and coupling with an example. (§9)
- [ ] I built `LibraryService` without copying. (§12.3)

### The question that opens Phase 4

The shop now sells e-books (with a download link) and physical goods (with a weight). The quick fix is to copy `Product.java` into `DigitalProduct.java` and add one field. Now `id`, `name`, `price` and all their getters exist twice. Marketing then asks for a `discountPrice` on **every** product by Friday: that is six separate edits across two files, and the copies will drift apart.

**How can two classes share everything they have in common, and each add only what makes it different?**

**Next: Phase 4, Inheritance.**
