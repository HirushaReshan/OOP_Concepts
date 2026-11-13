# Phase 4: Inheritance

**Module:** 5COSC019W Object Oriented Programming
**Before this:** Phase 3 (lists, maps, streams, the service class).
**Goal:** by the end you can write a child class that `extends` a parent, pass values up with `super(...)`, decide between IS-A and HAS-A, and draw the hierarchy in UML.
**Time needed:** about 2 hours.

**Built from your files:** Lecture 4 (Inheritance: Reuse Without Repetition), Tutorial 4 (Building the Product Hierarchy).

> **Why this phase matters most for the exam.** Lecture 6's mock briefing describes Task 1 (18 marks) as *"The file exists. Add a field, a constructor calling `super(...)`, and its summary"*, and Task 2 (22 marks) as *"You make the file this time: a field, two constructors and its own `getSummary()`."* That is **40 marks of subclass writing**. This phase and Phase 5 are the two to know best.

---

## Where we are on the roadmap

| Phase | Title | Status |
| :-- | :-- | :-- |
| Pre-Phase 0 | Absolute Java Basics | Done |
| Phase 1 | Classes and Objects | Done |
| Phase 2 | Encapsulation and UML Class Diagrams | Done |
| Phase 3 | Collections and Generics | Done |
| **Phase 4** | **Inheritance** | **You are here** |
| Phase 5 | Polymorphism and Abstraction | Next |
| Phase 6 | Exceptions | |
| Phase 7 | Dependencies and Layered Design | |
| Phase 8 | Exam Consolidation | |

---

## 1. The problem: copy-paste code

The shop now sells e-books, which need a download link. The quick fix is to copy `Product.java`, rename it, and add a field:

```java
// Product.java
public class Product {
    private Long id;
    private String name;
    private double price;
    // constructors + getters + setters
}

// DigitalProduct.java : copy, paste, add a field
public class DigitalProduct {
    private Long id;              // copied
    private String name;          // copied
    private double price;         // copied
    private String downloadUrl;   // new
    // constructors + getters + setters, all copied again
}
```

It compiles, and it works. Lecture 4 points out that this is exactly why the mistake survives into real projects.

**Then the change request arrives:** *"Marketing wants a `discountPrice` field on every product, by Friday."* With two copies, that means the field, the getter and the setter, each written twice: **six edits in two files**. Add `PhysicalProduct` as a third copy and it is nine. Miss one and the classes quietly disagree.

There is a second, hidden problem. As far as Java knows, `Product` and `DigitalProduct` are **unrelated**. You cannot put both in one `List<Product>`.

The name for this smell is **duplicate code**, and the principle it breaks is **DRY** (Don't Repeat Yourself, Phase 2).

---

## 2. `extends`: one word removes the duplication

### 2.1 Parent and child

**Inheritance** lets one class take on everything another class has, and then add its own extras. You write it with the keyword `extends`.

```java
public class DigitalProduct extends Product {
    ...
}
```

Read it as: "a `DigitalProduct` is a `Product`, plus some extras."

| Term | Also called | Meaning | Example |
| :-- | :-- | :-- | :-- |
| **Superclass** | Parent class, base class | The general class that holds what everything has in common | `Product` |
| **Subclass** | Child class, derived class | The specialised class that adds what makes it different | `DigitalProduct` |

- **Analogy: family traits.** A child inherits eye colour and height from a parent without doing anything, and then has their own talents too.
- **Analogy: a car model.** "Car" has wheels, an engine and seats. "Electric car" **is a** car, gets all of that automatically, and adds a battery.

Lecture 4:

| The parent (`Product`) | The child (`DigitalProduct`) |
| :-- | :-- |
| Holds everything every product has in common | Declares only what makes it different |
| Knows nothing about its children (it existed first) | Automatically has `id`, `name`, `price`: no code needed |
| Changing it changes every child at once | Can be used anywhere a `Product` is expected |

### 2.2 The parent

We use the Phase 2/3 `Product` (Tutorial 4's version):

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

### 2.3 The child

```java
public class DigitalProduct extends Product {

    private String downloadUrl;
    private double fileSizeMB;

    public DigitalProduct() {}

    public DigitalProduct(Long id, String name, double price,
                          String downloadUrl, double fileSizeMB) {
        super(id, name, price);
        this.downloadUrl = downloadUrl;
        this.fileSizeMB = fileSizeMB;
    }

    public String getDownloadUrl() { return downloadUrl; }
    public double getFileSizeMB() { return fileSizeMB; }
}
```

| Line | What it means |
| :-- | :-- |
| `public class DigitalProduct extends Product {` | "`DigitalProduct` is a new class that inherits everything from `Product`." |
| `private String downloadUrl;` | A field **only** digital products have. |
| `private double fileSizeMB;` | Another. |
| *(no `id`, `name`, `price`)* | **Not written here.** They live in `Product.java` and are inherited. |
| `public DigitalProduct() {}` | The no-argument constructor, for Spring's Jackson (Phase 2, Section 8.2). |
| `public DigitalProduct(Long id, ..., double fileSizeMB) {` | The full constructor: all **five** values, the three shared ones and the two new ones. |
| `super(id, name, price);` | **Hand the three shared values up to `Product`'s constructor.** Section 3 explains this line in depth. |
| `this.downloadUrl = downloadUrl;` | Store the child's own values in the usual way. |
| `getDownloadUrl()`, `getFileSizeMB()` | Getters for the child's own fields only. |
| *(no `getName()`, `getPrice()`)* | Inherited. Not written again. |

Lecture 4: the file shrank from about 40 lines to about 15, and one file, `Product.java`, now owns the shared knowledge. The `discountPrice` change becomes **one edit in one file**.

### 2.4 Inherited methods just work

```java
DigitalProduct ebook = new DigitalProduct(
        1L, "Clean Code", 29.99, "/downloads/clean-code.pdf", 4.2);

System.out.println(ebook.getName());          // Clean Code    (defined in Product)
System.out.println(ebook.getPrice());         // 29.99         (defined in Product)
System.out.println(ebook.getDownloadUrl());   // /downloads/clean-code.pdf
System.out.println(ebook.getFileSizeMB());    // 4.2
```

`getName()` was never written in `DigitalProduct`. Java looks in `DigitalProduct` first, does not find it, then looks in the parent `Product` and finds it there. There is **only one copy** of the code, in `Product`.

### 2.5 What the child gets, and what it does not

Lecture 4's table:

| Inherited | Not inherited |
| :-- | :-- |
| `public` and `protected` fields and methods | **Constructors: never inherited, ever** |
| Default (package) members, if in the same package | `private` fields cannot be **used directly** (they exist, but only through getters) |
| The parent's whole public face: `getName()`, `getPrice()`... | `private` methods |

The `private` row needs care. A `DigitalProduct` object **does** contain an `id`, a `name` and a `price`: they are part of the object. But code written **inside** `DigitalProduct` cannot touch them by name, because `private` means "only inside `Product`". Section 4 covers this.

### 2.6 Single inheritance

Lecture 4, quick check Q3: *"Can a Java class extend two parent classes at once?"* **No.** Java allows **single inheritance only**: one `extends` per class. (Phase 5 shows how interfaces work around this.)

```java
public class Thing extends Product, Course { }   // COMPILE ERROR
```

A chain is allowed: `C extends B`, `B extends A`. Then `C` gets everything from both `B` and `A`.

### 2.7 Every class already has a parent: `Object`

If you write no `extends`, Java quietly adds `extends Object`. **`Object`** is the class at the top of every Java hierarchy. It is why every object already has methods you never wrote, such as `toString()` and `equals()`. Phase 5 uses this.

---

## 3. `super(...)`: how a child sets up its parent part

### 3.1 Why it is needed

Constructors are **not** inherited. So `DigitalProduct` cannot simply use `Product`'s constructor as if it were its own. Yet the `id`, `name` and `price` fields are `private` to `Product`, so `DigitalProduct`'s constructor cannot assign them itself either.

The solution: the child's constructor **calls** the parent's constructor and lets the parent fill in its own fields.

**`super(...)`** means "run my parent's constructor with these values".

- **Analogy:** building a house extension. Before you can build the new room, the foundations of the main house must be laid. `super(...)` lays the foundations; then the child adds its own room.

### 3.2 What happens on `new`

For `new DigitalProduct(1L, "E-book", 9.99, "/dl/1", 4.2)`, Lecture 4's sequence:

| Step | What happens |
| :-- | :-- |
| 1 | Java creates the object and starts running `DigitalProduct`'s constructor. |
| 2 | `super(id, name, price)` runs **first**. Control jumps up to `Product`'s constructor, which sets `id`, `name`, `price`. |
| 3 | `Product`'s constructor finishes and returns. The inherited half of the object is ready. |
| 4 | Back in `DigitalProduct`'s constructor: `downloadUrl` and `fileSizeMB` are set. The object is complete. |

```
 new DigitalProduct(1L, "E-book", 9.99, "/dl/1", 4.2)
        |
        v
 DigitalProduct constructor
        |   super(1L, "E-book", 9.99) ------> Product constructor
        |                                      this.id = 1
        |                                      this.name = "E-book"
        |                                      this.price = 9.99
        |   <--------------------------------- returns
        |   this.downloadUrl = "/dl/1"
        |   this.fileSizeMB = 4.2
        v
 finished object: id, name, price, downloadUrl, fileSizeMB
```

### 3.3 The rule: `super(...)` must be the first line

```java
public DigitalProduct(Long id, String name, double price,
                      String downloadUrl, double fileSizeMB) {
    this.downloadUrl = downloadUrl;
    super(id, name, price);              // COMPILE ERROR on most Java versions
}
```

The parent must be set up **before** anything else happens. Put `super(...)` on the first line of the constructor body, always.

### 3.4 What if you forget `super(...)`?

If you write no `super(...)` at all, **Java silently inserts `super();`**, the no-argument version, as the first line.

That causes one of two situations:

| Parent has a no-arg constructor? | Result |
| :-- | :-- |
| **No** | **Compile error.** IntelliJ says *"There is no default constructor available in 'Product'"*; the command-line compiler says *"constructor Product in class Product cannot be applied to given types; required: Long,String,double; found: no arguments"*. |
| **Yes** | It **compiles**, but the parent's empty constructor runs, so `id`, `name` and `price` stay `null`/`0.0`. **A silent bug.** |

The second row is the dangerous one, and it is easy to fall into because Spring classes always have a no-arg constructor. **In every full child constructor, write `super(...)` with the shared values.**

The child's own **no-arg** constructor (`public DigitalProduct() {}`) is fine without a `super(...)` line: the hidden `super();` calls `Product()`, which is exactly what Jackson wants.

### 3.5 Lecture 4's advice: two constructors in every class

| Constructor | Who uses it |
| :-- | :-- |
| No-arg | Spring's Jackson, to build an object from incoming JSON and then call setters. |
| Full | Your own code, and every subclass through `super(...)`. |

In the exam, when asked to write a subclass "with two constructors", this is what is meant.

---

## 4. Access inside a child class

### 4.1 `private` fields hit the encapsulation wall

```java
public class DigitalProduct extends Product {

    public double priceWithVat() {
        return price * 1.2;          // COMPILE ERROR: price has private access in Product
    }
}
```

Lecture 4: *"Inheritance does not defeat private. A subclass is still outside code."* The parent keeps control of its own data.

**The fix: use the inherited public getter.**

```java
public double priceWithVat() {
    return getPrice() * 1.2;         // works
}
```

Calling `getPrice()` with nothing in front means "call it on this object". It is short for `this.getPrice()`.

Lecture 4, Q1: *"Inside `DigitalProduct`, can you write `price = 0.0` directly?"* **No.** Use `getPrice()` / `setPrice(...)`, or make the field `protected` if the design genuinely needs it.

### 4.2 `protected`: the deliberate middle ground

```java
public class Product {
    protected double price;      // visible to subclasses
}
```

`protected` (UML symbol `#`) means: visible in the same package, **and** to every subclass anywhere. Now `DigitalProduct` could write `price * 1.2` directly.

Lecture 4's practical rule: **fields `private`, methods `public`, `protected` only when a subclass has a real need. Never make fields `public` to make inheritance easier.**

If an exam starter's parent class uses `protected` fields, you may use them directly in your subclass. If it uses `private` fields, you must use the getters.

### 4.3 The four levels, seen from a subclass

| Modifier | Can the child class use it? |
| :-- | :-- |
| `private` | No. Use the parent's getters. |
| default (none) | Only if both classes are in the same package. |
| `protected` | Yes, always. |
| `public` | Yes, and so can everyone else. |

---

## 5. A child fits where the parent is expected

### 5.1 Parent-typed variables

```java
Product p = ebook;      // compiles
```

Lecture 4, Q3: *"Why does `Product p = ebook;` compile?"* Because **a `DigitalProduct` IS-A `Product`**. The child fits anywhere the parent is expected.

- **Analogy:** a sign says "cars only". An electric car is a car, so it is allowed in.

The reverse does **not** compile:

```java
DigitalProduct d = new Product(2L, "Mug", 8.50);   // COMPILE ERROR: incompatible types
```

Not every product is a digital product. A sign saying "electric cars only" does not let in every car.

Through a `Product` variable, you can only call methods that `Product` has:

```java
Product p = ebook;
p.getName();           // fine: Product has getName()
p.getDownloadUrl();    // COMPILE ERROR: cannot find symbol
```

The object really is a `DigitalProduct`, but the **variable's type** is `Product`, and the compiler only allows what the variable's type promises. Phase 5 builds on this exact point.

### 5.2 One list, two types

This is the payoff (Lecture 4, Section 07):

```java
public class PhysicalProduct extends Product {

    private double weightKg;
    private String dimensions;

    public PhysicalProduct() {}

    public PhysicalProduct(Long id, String name, double price,
                           double weightKg, String dimensions) {
        super(id, name, price);
        this.weightKg = weightKg;
        this.dimensions = dimensions;
    }

    public double getWeightKg() { return weightKg; }
    public String getDimensions() { return dimensions; }
}
```

```java
List<Product> products = new ArrayList<>();

products.add(new DigitalProduct(1L, "E-book", 9.99, "/dl/1", 4.2));
products.add(new PhysicalProduct(2L, "Monitor", 249.99, 5.4, "60x40"));

for (Product p : products) {
    System.out.println(p.getId() + " " + p.getName() + " " + p.getPrice());
}
```

**Output:**

```
1 E-book 9.99
2 Monitor 249.99
```

One list typed `List<Product>` holds both, because both IS-A `Product`. No casting, no separate lists, and the Phase 3 service keeps working unchanged.

In Spring, `GET /products` now returns a JSON array where one object has `downloadUrl` and the other has `weightKg`, and both still show `id`, `name` and `price`, because Jackson calls the inherited getters too.

**But:** the loop can only call `Product` methods. It cannot ask each item to do something different according to its kind. That is what Phase 5 (polymorphism) solves.

---

## 6. IS-A or HAS-A?

### 6.1 The sentence test

Lecture 4 calls this *the most common way inheritance goes wrong.* Say the sentence out loud:

| Relationship | Sentence | Java |
| :-- | :-- | :-- |
| **IS-A** | "A `DigitalProduct` **is a** `Product`." Sounds true. | `extends` |
| **HAS-A** | "A `ShoppingCart` **has** `Product`s." | A **field** |

If "is a" sounds wrong ("a shopping cart is a product"?), do not use `extends`.

Lecture 4's vote:

| Pair | Verdict | Reason |
| :-- | :-- | :-- |
| `PhysicalProduct` → `Product` | IS-A | A product that happens to be shipped. |
| `ShoppingCart` → `Product` | HAS-A | A cart contains products; it is not one. |
| `Manager` → `Employee` | IS-A | Every manager is an employee with extra duties. |
| `Car` → `Engine` | HAS-A | A car owns an engine. |
| `OnlineCourse` → `Course` | IS-A | *"Exactly the shape of your mock lab exam."* |
| `Order` → `Customer` | HAS-A | An order refers to a customer; it is not one. |

### 6.2 The wrong way and the right way

```java
// WRONG: inheriting just to reuse code
public class ShoppingCart extends Product {
    private List<Product> items;
}
// A cart now has an id, a name and a price. None of them make sense.
```

```java
// RIGHT: the cart simply holds products
public class ShoppingCart {

    private List<Product> items = new ArrayList<>();

    public void addItem(Product product) {
        items.add(product);
    }

    public double getTotal() {
        double total = 0;
        for (Product p : items) {
            total += p.getPrice();
        }
        return total;
    }
}
```

Lecture 4's rule of thumb: **inheritance (IS-A) is for identity; HAS-A is for ownership. If you are only reaching for `extends` to avoid retyping a few methods, you want a field.**

### 6.3 Two kinds of HAS-A: composition and aggregation

In Java both are just a field. The question that separates them: **"If the whole is deleted, does the part go with it?"**

| | **Composition** (strong) | **Aggregation** (weak) |
| :-- | :-- | :-- |
| Meaning | The part belongs to the whole and dies with it | The part exists independently and can be shared |
| Examples | An `Order` and its `OrderLine`s; a `House` and its `Room`s | A `ShoppingCart` and its `Product`s; a `Department` and its `Lecturer`s |
| Who creates the part? | Usually the whole | Usually someone else |
| UML | Solid line, **filled diamond ◆** at the whole's end | Solid line, **hollow diamond ◇** at the whole's end |

- **Analogy:** a tree and its leaves (composition: cut down the tree, the leaves go too). A team and its players (aggregation: disband the team, the players still exist).

---

## 7. UML for inheritance and HAS-A

### 7.1 The hollow triangle

Inheritance is drawn as a **solid line with a hollow (unfilled) triangle**, and the triangle **sits at the parent end**. Lecture 4 warns that direction is "commonly marked wrong".

```
              +---------------------------+
              |          Product          |
              +---------------------------+
              | - id : Long               |
              | - name : String           |
              | - price : double          |
              +---------------------------+
              | + getId() : Long          |
              | + getName() : String      |
              | + getPrice() : double     |
              +---------------------------+
                           /_\            <-- hollow triangle at the PARENT
                            |
             +--------------+---------------+
             |                              |
+---------------------------+  +---------------------------+
|      DigitalProduct       |  |      PhysicalProduct      |
+---------------------------+  +---------------------------+
| - downloadUrl : String    |  | - weightKg : double       |
| - fileSizeMB : double     |  | - dimensions : String     |
+---------------------------+  +---------------------------+
| + DigitalProduct()        |  | + PhysicalProduct()       |
| + DigitalProduct(id : Long|  | + PhysicalProduct(id :    |
|   , name : String, price :|  |   Long, name : String,    |
|   double, downloadUrl :   |  |   price : double, weightKg|
|   String, fileSizeMB :    |  |   : double, dimensions :  |
|   double)                 |  |   String)                 |
| + getDownloadUrl() : String| | + getWeightKg() : double  |
| + getFileSizeMB() : double|  | + getDimensions() : String|
+---------------------------+  +---------------------------+
```

Rules (Lecture 4):

- Hollow triangle, **at the parent**.
- **Do not repeat inherited attributes in the child box.** The arrow already says they are there.
- Visibility symbols as before: `-` private, `+` public, `#` protected.

### 7.2 The diamonds and multiplicity

```
+------------+ 1        1..* +-------------+
|   Order    |◆--------------|  OrderLine  |      composition
+------------+               +-------------+

+--------------+ *     0..* +-----------+
| ShoppingCart |◇-----------|  Product  |      aggregation
+--------------+            +-----------+
```

The diamond sits at the **whole** (the class that holds the field). The numbers are **multiplicity**: how many of each.

| Multiplicity | Meaning |
| :-- | :-- |
| `1` | Exactly one |
| `0..1` | None or one |
| `*` or `0..*` | Any number, including none |
| `1..*` | One or more |

So "one `Order` has one or more `OrderLine`s".

### 7.3 Every arrow so far

| Relationship | Line | Head | Head sits at | Reads as |
| :-- | :-- | :-- | :-- | :-- |
| Inheritance | Solid | Hollow triangle | **Parent** | "is a" |
| Association | Solid | Open arrow | Held class | "has a" |
| Composition | Solid | Filled diamond | **Whole** | "owns, and the part dies with it" |
| Aggregation | Solid | Hollow diamond | **Whole** | "has, but the part lives on" |
| Dependency | **Dashed** | Open arrow | Used class | "uses" |

---

## 8. A second worked example: the mock-exam shape

Lecture 4 says `OnlineCourse → Course` is "exactly the shape of your mock lab exam." Here is that shape, written with everything above.

```java
public class Course {

    private Long id;
    private String title;
    private double fee;

    public Course() {}

    public Course(Long id, String title, double fee) {
        this.id = id;
        this.title = title;
        this.fee = fee;
    }

    public Long getId() { return id; }
    public String getTitle() { return title; }
    public double getFee() { return fee; }

    public void setId(Long id) { this.id = id; }
    public void setTitle(String title) { this.title = title; }
    public void setFee(double fee) { this.fee = fee; }
}
```

```java
public class OnlineCourse extends Course {

    private String platform;

    public OnlineCourse() {}

    public OnlineCourse(Long id, String title, double fee, String platform) {
        super(id, title, fee);
        this.platform = platform;
    }

    public String getPlatform() { return platform; }
    public void setPlatform(String platform) { this.platform = platform; }
}
```

```java
public class InPersonCourse extends Course {

    private String room;
    private int capacity;

    public InPersonCourse() {}

    public InPersonCourse(Long id, String title, double fee, String room, int capacity) {
        super(id, title, fee);
        this.room = room;
        this.capacity = capacity;
    }

    public String getRoom() { return room; }
    public int getCapacity() { return capacity; }
}
```

The recipe, every time:

1. `public class Child extends Parent {`
2. Declare **only the new** fields, `private`.
3. A no-arg constructor: `public Child() {}`.
4. A full constructor taking **parent's values first, then the new ones**.
5. First line: `super(parentValues...);`
6. Then `this.newField = newField;` for each new field.
7. Getters (and setters if needed) for the **new** fields only.

---

## 9. Common mistakes

| Mistake | What happens | Fix |
| :-- | :-- | :-- |
| Re-declaring `id`, `name`, `price` in the child | Two copies of each field; the getters read the parent's, so the child's stay empty. A nasty silent bug. | Declare only the new fields. |
| Forgetting `super(...)` when the parent has no no-arg constructor | Compile error. | `super(id, name, price);` as the first line. |
| Forgetting `super(...)` when the parent **does** have a no-arg constructor | Compiles; inherited fields stay `null`/`0`. | Always call the full `super(...)`. |
| `super(...)` not on the first line | Compile error. | Move it to the top. |
| Arguments to `super` in the wrong order | Compile error, or silently swapped values if types match. | Match the parent constructor's parameter order exactly. |
| Using a parent's `private` field directly | Compile error: "has private access". | Use the getter. |
| Using `extends` for HAS-A | A cart with a price. Nonsense design. | Use a field. |
| Two `extends` | Compile error. | One parent only. |
| Subclass with no no-arg constructor (in Spring) | POST with JSON fails. | Add `public Child() {}`. |
| UML triangle at the child end | Marked wrong. | Triangle at the **parent**. |
| Repeating inherited fields in the child's UML box | Marked wrong. | Only the new fields. |

---

## 10. Practice

### 10.1 Tutorial 4 warm-up: where does each piece live?

Sort each member into `Product`, `DigitalProduct` or `PhysicalProduct`. The test: **"Does every single product need this?"**

| # | Member | Your answer |
| :-- | :-- | :-- |
| 1 | `id : Long` | |
| 2 | `name : String` | |
| 3 | `downloadUrl : String` | |
| 4 | `price : double` | |
| 5 | `weightKg : double` | |
| 6 | `getName() : String` | |
| 7 | `fileSizeMB : double` | |
| 8 | `dimensions : String` | |
| 9 | `getPrice() : double` | |

**Answers:** `Product`: 1, 2, 4, 6, 9 (five members, the shared core). `DigitalProduct`: 3, 7. `PhysicalProduct`: 5, 8.

### 10.2 Quick checks (Lecture 4)

| # | Question | Answer |
| :-- | :-- | :-- |
| 1 | If `DigitalProduct extends Product`, how many files hold the `price` field? | One: `Product.java`. The child has `price` but does not declare it. |
| 2 | Which member of a parent class is never inherited? | The constructor. The child must call it with `super(...)`. |
| 3 | Can a class extend two parents? | No. Single inheritance only. |
| 4 | Inside `DigitalProduct`, can you write `price = 0.0`? | No: `price` is private to `Product`. Use the getter/setter, or `protected`. |
| 5 | Where does `getName()` live when called on a `DigitalProduct`? | In `Product`. One copy of the code. |
| 6 | Why does `Product p = ebook;` compile? | A `DigitalProduct` IS-A `Product`. |
| 7 | Why must `super(...)` come first? | The parent part of the object must be set up before the child adds to it. |

### 10.3 Predict: compile or not?

```java
Product a = new DigitalProduct(1L, "E-book", 9.99, "/dl", 2.0);    // A
DigitalProduct b = new Product(2L, "Mug", 8.50);                  // B
Product c = new PhysicalProduct(3L, "Desk", 120.0, 25.0, "1x2");  // C
System.out.println(a.getName());                                  // D
System.out.println(a.getDownloadUrl());                           // E
List<Product> list = new ArrayList<>();
list.add(a);                                                      // F
list.add(c);                                                      // G
```

| Line | Compiles? | Why |
| :-- | :-- | :-- |
| A | Yes | A child fits in a parent variable. |
| B | **No** | A plain `Product` is not a `DigitalProduct`. |
| C | Yes | Same as A. |
| D | Yes | `Product` has `getName()`. Prints `E-book`. |
| E | **No** | The variable's type is `Product`, which has no `getDownloadUrl()`. |
| F | Yes | A `DigitalProduct` is a `Product`. |
| G | Yes | A `PhysicalProduct` is a `Product`. |

### 10.4 Build it: `Vehicle`

Write:

- `Vehicle` with private `registration` (`String`), `make` (`String`) and `dailyRate` (`double`); a no-arg and a full constructor; getters.
- `Car extends Vehicle` adding `seats` (`int`).
- `Van extends Vehicle` adding `loadCapacityKg` (`double`) and a method `isHeavyDuty()` returning true if the capacity is above 1000.
- In `main`, put one `Car` and one `Van` in a `List<Vehicle>` and print each registration and daily rate.

Try for 20 minutes.

```java
public class Vehicle {

    private String registration;
    private String make;
    private double dailyRate;

    public Vehicle() {}

    public Vehicle(String registration, String make, double dailyRate) {
        this.registration = registration;
        this.make = make;
        this.dailyRate = dailyRate;
    }

    public String getRegistration() { return registration; }
    public String getMake() { return make; }
    public double getDailyRate() { return dailyRate; }
}
```

```java
public class Car extends Vehicle {

    private int seats;

    public Car() {}

    public Car(String registration, String make, double dailyRate, int seats) {
        super(registration, make, dailyRate);
        this.seats = seats;
    }

    public int getSeats() { return seats; }
}
```

```java
public class Van extends Vehicle {

    private double loadCapacityKg;

    public Van() {}

    public Van(String registration, String make, double dailyRate, double loadCapacityKg) {
        super(registration, make, dailyRate);
        this.loadCapacityKg = loadCapacityKg;
    }

    public double getLoadCapacityKg() { return loadCapacityKg; }

    public boolean isHeavyDuty() {
        return loadCapacityKg > 1000;
    }
}
```

```java
List<Vehicle> fleet = new ArrayList<>();
fleet.add(new Car("CAB-1234", "Toyota", 45.0, 5));
fleet.add(new Van("LV-5678", "Ford", 80.0, 1200.0));

for (Vehicle v : fleet) {
    System.out.println(v.getRegistration() + " " + v.getDailyRate());
}
```

```
CAB-1234 45.0
LV-5678 80.0
```

Draw the UML: `Vehicle` at the top, `Car` and `Van` below, one hollow triangle at `Vehicle`.

---

## 11. Glossary

| Term | Plain-English meaning |
| :-- | :-- |
| **Aggregation** | A weak HAS-A: the part exists independently (hollow diamond). |
| **Base class** | Another word for superclass. |
| **Composition** | A strong HAS-A: the part belongs to the whole and dies with it (filled diamond). |
| **Derived class** | Another word for subclass. |
| **Duplicate code** | The same code written in several places; breaks DRY. |
| **`extends`** | The keyword that makes one class inherit from another. |
| **Generalisation** | UML's word for inheritance. |
| **HAS-A** | Ownership: use a field. |
| **Hierarchy** | A parent with its children (and their children). |
| **Inheritance** | A class taking on all non-private members of another class and adding its own. |
| **IS-A** | Identity: the child is a kind of the parent. Use `extends`. |
| **Multiplicity** | UML numbers showing how many: `1`, `0..1`, `*`, `1..*`. |
| **`Object`** | The class at the top of every Java hierarchy. |
| **`protected`** | Visible in the same package and to all subclasses (`#`). |
| **Single inheritance** | A Java class may extend only one parent. |
| **Subclass / child** | The specialised class that extends another. |
| **`super(...)`** | Calls the parent's constructor. Must be the first line. |
| **Superclass / parent** | The general class being extended. |

---

## 12. One-page cheat sheet

```java
// ---------- the parent ----------
public class Parent {
    private Long id;
    private String name;

    public Parent() {}                                     // for Jackson
    public Parent(Long id, String name) {                  // for your code and super(...)
        this.id = id;
        this.name = name;
    }
    public Long getId() { return id; }
    public String getName() { return name; }
}

// ---------- the child ----------
public class Child extends Parent {                        // IS-A, one parent only
    private String extra;                                  // ONLY the new field(s)

    public Child() {}                                      // hidden super() calls Parent()

    public Child(Long id, String name, String extra) {     // parent's values first
        super(id, name);                                   // FIRST line
        this.extra = extra;                                // then the new ones
    }

    public String getExtra() { return extra; }             // getters for new fields only

    public String describe() {
        return getName() + " / " + extra;                  // parent's private field: use getter
    }
}

// ---------- using them ----------
Parent p = new Child(1L, "A", "x");                       // child fits in parent variable
List<Parent> all = new ArrayList<>();                      // one list, many child types
```

| Remember | |
| :-- | :-- |
| `extends` = IS-A | A field = HAS-A |
| Constructors are never inherited | Child calls `super(...)`, first line |
| Forgetting `super(...)` inserts `super()` | Compile error, or silent empty fields |
| Child cannot touch parent's `private` fields | Use getters, or `protected` |
| Never re-declare inherited fields | Only the new ones |
| A child fits in a parent variable | Not the other way round |
| Parent variable only allows parent methods | Even if the object is a child |
| UML: hollow triangle at the parent | Diamonds at the whole |

---

## 13. Am I ready for Phase 5?

- [ ] I can explain why copy-paste classes are a maintenance problem. (§1)
- [ ] I can write a subclass with `extends` that declares only its new fields. (§2)
- [ ] I can write a full child constructor with `super(...)` on the first line, and a no-arg one. (§3)
- [ ] I can explain what happens if `super(...)` is forgotten, in both cases. (§3.4)
- [ ] I know why a child cannot use the parent's `private` fields, and the two fixes. (§4)
- [ ] I can explain why `Product p = new DigitalProduct(...)` compiles, and why `p.getDownloadUrl()` does not. (§5)
- [ ] I can apply the IS-A / HAS-A sentence test. (§6)
- [ ] I can tell composition from aggregation. (§6.3)
- [ ] I can draw an inheritance hierarchy with the triangle the right way round. (§7)
- [ ] I built `Vehicle`, `Car` and `Van` without copying. (§10.4)

### The question that opens Phase 5

The list holds digital and physical products together. Now we want to print a summary for each: the link for digital ones, the weight for physical ones. The obvious approach asks every object what it is:

```java
for (Product p : products) {
    if (p instanceof DigitalProduct) {
        DigitalProduct d = (DigitalProduct) p;
        System.out.println("Digital: " + d.getName() + " | " + d.getDownloadUrl());
    } else if (p instanceof PhysicalProduct) {
        PhysicalProduct ph = (PhysicalProduct) p;
        System.out.println("Physical: " + ph.getName() + " | " + ph.getWeightKg() + "kg");
    }
}
```

It works. But every new product type means another branch, in every loop like this one, everywhere in the program. **What if each object could simply answer for itself?**

**Next: Phase 5, Polymorphism and Abstraction.**
