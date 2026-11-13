# Phase 5: Polymorphism and Abstraction

**Module:** 5COSC019W Object Oriented Programming
**Before this:** Phase 4 (`extends`, `super(...)`, one list holding several child types).
**Goal:** by the end you can override a method so each subclass answers differently, explain which version runs and why, write abstract classes and interfaces, and use `instanceof` and casting safely.
**Time needed:** about 2.5 hours.

**Built from your files:** Lecture 5 (Polymorphism: One Call, Many Answers), Tutorial 5 (Making the Hierarchy Behave).

> **Why this phase matters most for the exam.** Task 2 is literally titled *"Applied OOP and polymorphism"* (22 marks). In the mock, the parent class is **abstract**, and both subclass tasks end with "and its `getSummary()`". Section B also checks that *"the two subclasses' summaries read differently"*. Lecture 5 calls a missing or misspelled override *"the single most common silent bug in the assessed lab practical."*

---

## Where we are on the roadmap

| Phase | Title | Status |
| :-- | :-- | :-- |
| Pre-Phase 0 | Absolute Java Basics | Done |
| Phase 1 | Classes and Objects | Done |
| Phase 2 | Encapsulation and UML Class Diagrams | Done |
| Phase 3 | Collections and Generics | Done |
| Phase 4 | Inheritance | Done |
| **Phase 5** | **Polymorphism and Abstraction** | **You are here** |
| Phase 6 | Exceptions | Next |
| Phase 7 | Dependencies and Layered Design | |
| Phase 8 | Exam Consolidation | |

---

## 1. The problem: asking every object what it is

Phase 4 ended with one `List<Product>` holding digital and physical products. To print a summary that differs by type, the obvious code asks each object what it is:

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

Two new tools appear here, explained properly in Section 8:

| Piece | Meaning |
| :-- | :-- |
| `p instanceof DigitalProduct` | A yes/no question: "is the object `p` refers to really a `DigitalProduct`?" |
| `(DigitalProduct) p` | A **cast**: "treat `p` as a `DigitalProduct`, so I can call its extra methods." |

It works. Lecture 5 lists what is wrong with it:

| Problem | Why it hurts |
| :-- | :-- |
| The loop knows about every type | Add `BundleProduct` and you must edit this loop, **and every other loop like it** in the program. |
| A cast in every branch | Get one wrong and it crashes at runtime; the compiler cannot help. |
| Nothing warns you if you forget a branch | The new type silently prints nothing. |

Lecture 5 names the smell **type checking**, and the principle it breaks the **Open/Closed Principle**: code should be *open for extension, closed for modification*. Adding a new type should mean **adding a file**, not editing five.

**Key takeaway (Lecture 5): stop asking objects what they are. Ask them to do something, and let each type answer in its own way.**

- **Analogy:** a teacher says "everyone, introduce yourself". She does not ask each pupil "are you from Kandy? then say this; are you from Galle? then say that". Each pupil simply introduces themselves in their own words. One instruction, many different answers.

That is **polymorphism**: from Greek, "many forms". One method call, many different behaviours, chosen by the object.

---

## 2. Method overriding

### 2.1 Give the parent a method

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

    public String getSummary() {
        return name + " - £" + price;
    }
}
```

Every product can now summarise itself in a general way.

### 2.2 Each child replaces it

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

    @Override
    public String getSummary() {
        return "Digital: " + getName() + " | " + getDownloadUrl();
    }
}
```

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

    @Override
    public String getSummary() {
        return "Physical: " + getName() + " | " + weightKg + "kg";
    }
}
```

**Overriding** means: a child class writes its own version of a method it inherited, with **exactly the same name and parameters**, so that its version is used instead of the parent's.

- **Analogy:** a family recipe for curry, passed down. Your branch of the family keeps the name "Grandma's curry" but adds more chilli. When someone in your house asks for "Grandma's curry", they get your version.

The overriding method, line by line:

| Piece | What it means |
| :-- | :-- |
| `@Override` | An **annotation** (a label starting with `@` that gives the compiler extra information). It says: "this method is meant to replace one from my parent." Section 2.3 explains why it matters so much. |
| `public String getSummary()` | **Exactly** the same header as in `Product`. |
| `"Digital: " + getName()` | `getName()`, not `name`: the field is `private` to `Product` (Phase 4's rule still holds). |
| `getDownloadUrl()` | The child's own getter. Inside `PhysicalProduct`, `weightKg` can be used directly because it is the child's own field. |

### 2.3 What `@Override` actually protects you from

```java
public class DigitalProduct extends Product {

    @Override
    public String getsummary() {          // lower-case 's'
        return "Digital: " + getName();
    }
}
```

```
error: method does not override or implement a method from a supertype
```

(IntelliJ words it *"Method does not override method from its superclass"*.)

`@Override` is not required for the code to compile. **But without it, the typo compiles silently.** `getsummary()` becomes a brand-new method that nobody ever calls, `Product.getSummary()` keeps running, and every digital product shows the generic summary. No error. No warning.

Lecture 5: *"This is the single most common silent bug in the assessed lab practical."*

> **Exam rule: write `@Override` on every overriding method, every time.** It turns a silent bug into a compile error you can see and fix. Even better, let IntelliJ write the header: **Ctrl + O** lists the methods you can override and generates a correct signature.

### 2.4 The four rules of a valid override

| Rule | Detail |
| :-- | :-- |
| **Same name** | Character for character. `getSummary` ≠ `getsummary`. |
| **Same parameters** | Same types, same order. Different parameters make it an **overload** (Section 2.5), not an override. |
| **Same return type** | (Or a subclass of it.) |
| **Access never weaker** | A `public` method cannot be overridden as `private` or default. You may widen, never narrow. |

Methods that **cannot** be overridden: `private` methods (the child cannot see them), `static` methods, and methods marked `final`.

### 2.5 Overriding is not overloading

Both involve "same name", so they are easy to confuse.

| | **Overriding** | **Overloading** (Phase 2) |
| :-- | :-- | :-- |
| Where | Child class replaces a parent's method | Usually in the same class |
| Name | Same | Same |
| Parameters | **Same** | **Different** |
| Who decides which runs | The **object**, at runtime | The **arguments**, at compile time |
| Example | `getSummary()` in parent and child | `getSummary()` and `getSummary(boolean verbose)` |

Lecture 5's warning: *change the parameter list on an "override" and you have silently written an overload. The parent's version keeps running.* `@Override` catches this too.

### 2.6 Reusing the parent's version: `super.method()`

A child can **add to** the parent's version instead of replacing it entirely. `super.getSummary()` calls the parent's version from inside the child:

```java
@Override
public String getSummary() {
    return super.getSummary() + " (digital download)";
}
```

| Piece | Meaning |
| :-- | :-- |
| `super.getSummary()` | Run `Product`'s version and use its result. |
| `+ " (digital download)"` | Then add the child's extra. |

Result: `E-book - £9.99 (digital download)`. Compare: `super(...)` with brackets calls the parent's **constructor**; `super.name()` with a dot calls one of the parent's **methods**.

### 2.7 The most common override of all: `toString()`

Every class inherits `toString()` from `Object` (Phase 4, Section 2.7). Java calls it automatically whenever an object is printed or joined to text. The inherited version prints something unhelpful like `Product@1b6d3586`. Override it:

```java
@Override
public String toString() {
    return "Product{id=" + getId() + ", name=" + getName() + ", price=" + getPrice() + "}";
}
```

Now `System.out.println(p)` prints `Product{id=1, name=Mug, price=8.5}`. This is very useful for debugging in the exam.

---

## 3. Runtime dispatch: the object decides

### 3.1 Every object has two types

```java
Product p = new DigitalProduct(2L, "E-book", 9.99, "/dl/2", 4.2);
```

| | **Reference type** | **Object type** |
| :-- | :-- | :-- |
| What it is | The type the **variable** is declared as, **left** of `=` | The type actually built with `new`, **right** of `=` |
| Here | `Product` | `DigitalProduct` |
| Fixed when? | At compile time | At runtime |
| Controls | **Which methods you are allowed to call** | **Which version of the method actually runs** |

**Lecture 5's key takeaway: the reference type decides what you are allowed to call. The object type decides what actually runs.**

- `p.getSummary()` **compiles** (because `Product` has `getSummary()`), and runs **`DigitalProduct`'s** version (because the object is a `DigitalProduct`).
- `p.getDownloadUrl()` **does not compile**, because `Product` has no such method, even though the object really has one.

- **Analogy: a TV remote.** A universal remote (the reference type, `Product`) only has the buttons every TV has: on, off, volume. Point it at a fancy smart TV (the object type, `DigitalProduct`) and press "on": the smart TV turns on **in its own way**, with its own start-up screen. But the universal remote has no "open Netflix" button, even though this TV can do it.

### 3.2 What the JVM does, step by step (Lecture 5)

For `p.getSummary()` where `p` holds a `DigitalProduct`:

| Step | What happens |
| :-- | :-- |
| 1 | **Compile time:** the compiler checks the reference type. `Product` declares `getSummary()`, so the call is legal. It does **not** choose which version. |
| 2 | The compiled code says "call `getSummary()` on whatever this object turns out to be". |
| 3 | **Runtime:** the JVM looks at the real object. It is a `DigitalProduct`. |
| 4 | The most specific version wins: `DigitalProduct.getSummary()` runs. If `DigitalProduct` had no override, the JVM would walk **up** to `Product`'s version. |

Choosing the method at runtime from the real object is called **runtime dispatch** (also **dynamic binding** or **dynamic dispatch**).

### 3.3 Predict the output (Lecture 5)

```java
Product a = new Product(1L, "Mug", 8.50);
Product b = new DigitalProduct(2L, "E-book", 9.99, "/dl/2", 4.2);
Product c = new PhysicalProduct(3L, "Monitor", 249.99, 5.4, "60x40");

System.out.println(a.getSummary());
System.out.println(b.getSummary());
System.out.println(c.getSummary());
```

**Output:**

```
Mug - £8.5
Digital: E-book | /dl/2
Physical: Monitor | 5.4kg
```

Three variables of the same type. Three identical calls. Three different answers. Nothing in the calling code changed; **the objects decided.**

### 3.4 One loop, many voices

The whole `instanceof` chain from Section 1 becomes:

```java
public List<String> getAllSummaries() {
    List<String> summaries = new ArrayList<>();
    for (Product p : products) {
        summaries.add(p.getSummary());
    }
    return summaries;
}
```

Or as a stream (Phase 3):

```java
public List<String> getAllSummaries() {
    return products.stream()
            .map(Product::getSummary)
            .toList();
}
```

Tutorial 5: *"Your loop body should be one line. If it is longer than one line, you have written something you do not need."*

| Before (`instanceof` chain) | After (one polymorphic call) |
| :-- | :-- |
| Twelve lines of branching | One line |
| A cast in every branch | No casts |
| Adding a type means editing this loop | Adding a type means adding one file |
| A missed branch fails silently | A missing override falls back to the parent's version, deliberately |

Add a `BundleProduct` tomorrow with its own `getSummary()`, and `getAllSummaries()` does not change by a single character. Tutorial 5's stretch question: *"How many lines of `getAllSummaries()` did you have to change? If the answer is not zero, something in your design is not doing its job."*

### 3.5 Lecture 5 quick checks

| Question | Answer |
| :-- | :-- |
| `Product p = new DigitalProduct(...)`. Which `getSummary()` runs? | `DigitalProduct`'s. The object type chooses. |
| On that same variable, can you call `p.getDownloadUrl()`? | No. The reference type is `Product`, which has no such method. |
| `PhysicalProduct` has no `getSummary()` of its own. What prints? | `Product`'s version. The JVM walks up the hierarchy until it finds one. |
| Delete `@Override`: does it still compile? | Yes. But you lose the check, and a typo becomes a silent bug. |
| Parent has `getSummary()`, child has `getSummary(boolean verbose)`. Override or overload? | Overload. The child now has both, and still inherits the parent's version. |

---

## 4. Abstract classes: a parent that refuses to be created

### 4.1 What is a "plain product"?

`new Product(1L, "Mug", 8.50)` compiles today. But the shop has nothing that is neither digital nor physical. A plain `Product` is meaningless, and its generic `getSummary()` is a placeholder nobody should see.

### 4.2 `abstract` in two places

```java
public abstract class Product {

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

    public abstract String getSummary();
}
```

| Piece | What it means |
| :-- | :-- |
| `public abstract class Product` | An **abstract class**: it **cannot be created with `new`**. It exists only to be a parent. |
| Fields, constructors, getters | Still allowed, and still inherited. |
| `public abstract String getSummary();` | An **abstract method**: a header with **no body**. It ends in a **semicolon**, not `{ }`. |

An abstract method says: "every product **must** be able to summarise itself, but I cannot say how; each child must decide."

- **Analogy:** "Animal" is an abstract idea. You have never met a plain "animal", only dogs, cats and birds. Every animal must be able to "make a sound", but there is no general animal sound. Each kind supplies its own.

A class that is **not** abstract is called **concrete**.

### 4.3 What happens now

```java
Product p = new Product(1L, "Mug", 8.50);
```

```
error: Product is abstract; cannot be instantiated
```

And if a concrete child forgets to write the abstract method:

```java
public class PhysicalProduct extends Product {
    // no getSummary()
}
```

```
error: PhysicalProduct is not abstract and does not override abstract method getSummary() in Product
```

The compiler now **forces** every subclass to answer. Nothing can be forgotten.

You can still **use** `Product` as a type: `Product p = new DigitalProduct(...)` and `List<Product>` both work exactly as before. You simply cannot build a plain one.

### 4.4 What `abstract` buys you (Lecture 5)

| Feature | Meaning |
| :-- | :-- |
| Cannot be instantiated | `new Product(...)` stops compiling. The type still works for variables, lists and parameters. |
| State is still allowed | It can hold `id`, `name`, `price` and a constructor to set them. |
| Shared code stays shared | `getName()` is written once and inherited. |
| Abstract methods force every subclass | Forget one and the subclass will not compile. |

Rules to remember:

- A class with **any** abstract method must itself be declared `abstract`.
- An abstract class **can** have constructors. Children call them with `super(...)`.
- An abstract class does **not** have to contain any abstract methods.
- A child of an abstract class must override **every** abstract method, or be declared abstract itself.

> **This is the exam's shape.** Lecture 6's mock briefing: *"The abstract parent class, the repository and ErrorResponse: read them, do not edit them."* Expect an abstract parent with an abstract `getSummary()` (or similar). Your subclasses must `extends` it, call `super(...)`, and override every abstract method with `@Override`.

---

## 5. Interfaces: what a class can do

### 5.1 One parent, many contracts

Phase 4: a class can `extends` only **one** parent. But a digital product might need several unrelated abilities: being discountable, being searchable, being shippable...

An **interface** is a **contract**: a named list of methods a class promises to provide. A class signs the contract with the keyword **`implements`**, and may implement **as many interfaces as it likes**.

| | `extends` | `implements` |
| :-- | :-- | :-- |
| Used with | A class | An interface |
| How many | Exactly one | Any number |
| Answers the question | **What is this thing?** | **What can this thing do?** |

```java
public class DigitalProduct extends Product implements Discountable, Searchable { ... }
```

- **Analogy:** your **identity** is "a student" (one thing you are: `extends`). Your **abilities** are licences and certificates: driving licence, first-aid certificate, swimming badge (many things you can do: `implements`). Anyone who checks "can you drive?" only cares about the licence, not who you are.

### 5.2 Writing an interface

```java
public interface Discountable {
    void applyDiscount(double rate);
    double getDiscountedPrice();
}
```

| Piece | What it means |
| :-- | :-- |
| `public interface Discountable` | `interface` instead of `class`. Interface names are often adjectives ending in `-able`. |
| `void applyDiscount(double rate);` | A method header with no body: part of the contract. |
| No `public` / `abstract` written on the methods | Interface methods are automatically `public` and `abstract`. |
| No fields, no constructor | An interface holds no object state. You cannot write `new Discountable()`. |

### 5.3 Implementing it: same contract, different rules

From Lecture 5:

```java
public class DigitalProduct extends Product implements Discountable {

    private String downloadUrl;
    private double fileSizeMB;
    private double discountedPrice;

    public DigitalProduct(Long id, String name, double price,
                          String downloadUrl, double fileSizeMB) {
        super(id, name, price);
        this.downloadUrl = downloadUrl;
        this.fileSizeMB = fileSizeMB;
        this.discountedPrice = price;
    }

    public String getDownloadUrl() { return downloadUrl; }

    @Override
    public String getSummary() {
        return "Digital: " + getName() + " | " + downloadUrl;
    }

    @Override
    public void applyDiscount(double rate) {
        discountedPrice = getPrice() * (1 - rate);
    }

    @Override
    public double getDiscountedPrice() {
        return discountedPrice;
    }
}
```

```java
public class PhysicalProduct extends Product implements Discountable {

    private double weightKg;
    private String dimensions;
    private double discountedPrice;

    public PhysicalProduct(Long id, String name, double price,
                           double weightKg, String dimensions) {
        super(id, name, price);
        this.weightKg = weightKg;
        this.dimensions = dimensions;
        this.discountedPrice = price;
    }

    public double getWeightKg() { return weightKg; }

    @Override
    public String getSummary() {
        return "Physical: " + getName() + " | " + weightKg + "kg";
    }

    @Override
    public void applyDiscount(double rate) {
        double capped = Math.min(rate, 0.20);
        discountedPrice = getPrice() * (1 - capped);
    }

    @Override
    public double getDiscountedPrice() {
        return discountedPrice;
    }
}
```

| Line | What it means |
| :-- | :-- |
| `extends Product implements Discountable` | `extends` first, then `implements`. |
| `@Override` on interface methods | Also correct and recommended: it checks you matched the contract. |
| `getPrice() * (1 - rate)` | A rate of `0.10` means 10% off: price × 0.90. |
| `Math.min(rate, 0.20)` | `Math.min` returns the smaller of two numbers. Physical products never get more than 20% off. |

Same contract, two different rules. A class that `implements` an interface but does not write all its methods will not compile (unless the class is abstract).

```java
DigitalProduct ebook = new DigitalProduct(1L, "E-book", 10.0, "/dl/1", 2.0);
PhysicalProduct desk = new PhysicalProduct(2L, "Desk", 100.0, 25.0, "1x2");

ebook.applyDiscount(0.50);
desk.applyDiscount(0.50);

System.out.println(ebook.getDiscountedPrice());   // 5.0
System.out.println(desk.getDiscountedPrice());    // 80.0
```

### 5.4 An interface is a type too

```java
List<Discountable> items = new ArrayList<>();
items.add(ebook);
items.add(desk);

for (Discountable d : items) {
    d.applyDiscount(0.10);
}
```

A variable or list can be typed by the interface. It can then hold **any** object whose class implements it, even classes with no common parent. This is polymorphism through interfaces, and it is exactly the idea behind `List<Product> x = new ArrayList<>()` in Phase 3 (`List` is an interface). Phase 7 builds on it.

### 5.5 `default` methods (good to recognise)

Modern Java lets an interface give a method a body, marked `default`. Implementing classes get it for free and may override it.

```java
public interface Discountable {
    void applyDiscount(double rate);
    double getDiscountedPrice();

    default boolean isOnSale() {
        return true;
    }
}
```

Your lectures treat interfaces as "no method bodies" (classic Java). If you see `default` in exam code, you now know what it means.

---

## 6. Abstract class or interface?

| | **Abstract class** (what it **is**) | **Interface** (what it **can do**) |
| :-- | :-- | :-- |
| Fields and constructors | Yes: real, shared state | No instance fields, no constructor |
| Method bodies | A mix of abstract and fully written methods | Classically none (`default` methods aside) |
| How many per class | Only one (`extends`) | Any number (`implements`) |
| Choose it when | Subclasses genuinely share data and code | Unrelated classes need the same capability |

Lecture 5's vote:

| Need | Choice | Reason |
| :-- | :-- | :-- |
| Shared `id`, `name` and `price` | Abstract class | Real shared state. Only a class can hold fields. |
| Can be discounted | Interface | A capability, not an identity. A `Voucher` could be discountable too. |
| Must summarise itself | Abstract class | An abstract method on the existing parent forces it. |
| Searchable by keyword | Interface | Products, courses and customers could all offer it. |
| Shared constructor logic | Abstract class | Interfaces have no constructors. If subclasses need `super(...)`, you need a class. |
| Unrelated classes, same method | Interface | Do not invent a common parent just to share a method. |

---

## 7. The four pillars, all together

You have now used all four pillars of OOP your module names:

| Pillar | One-line meaning | Where you learned it | Java tools |
| :-- | :-- | :-- | :-- |
| **Encapsulation** | Hide the data; expose safe methods | Phase 2 | `private`, getters, setters, validation |
| **Inheritance** | Reuse a parent's fields and methods | Phase 4 | `extends`, `super(...)` |
| **Polymorphism** | One call, different behaviour per object | Phase 5 | Overriding, `@Override`, runtime dispatch |
| **Abstraction** | Show what something does, hide how; forbid meaningless objects | Phase 5 | `abstract` classes and methods, interfaces |

A good exam or report answer names the pillar **and** points to the code that shows it.

---

## 8. `instanceof` and casting, done safely

### 8.1 Casting down

**Upcasting** (child → parent) is automatic and always safe: `Product p = ebook;`.

**Downcasting** (parent reference → child type) needs an explicit cast, and can fail:

```java
Product b = new DigitalProduct(2L, "E-book", 9.99, "/dl/2", 4.2);
Product a = new PhysicalProduct(3L, "Desk", 120.0, 25.0, "1x2");

String url1 = ((DigitalProduct) b).getDownloadUrl();   // fine: b really is a DigitalProduct
String url2 = ((DigitalProduct) a).getDownloadUrl();   // compiles, then crashes
```

```
Exception in thread "main" java.lang.ClassCastException: class PhysicalProduct cannot be cast to class DigitalProduct
```

| Piece | Meaning |
| :-- | :-- |
| `(DigitalProduct) b` | "Trust me, `b` is a `DigitalProduct`." |
| Extra brackets around `((DigitalProduct) b)` | So the cast happens **before** `.getDownloadUrl()`. |
| Why the second compiles | The compiler cannot know what `a` really holds; it trusts the cast. |
| Why it crashes | At runtime the JVM checks, finds a `PhysicalProduct`, and refuses. |

### 8.2 Check first with `instanceof`

```java
if (a instanceof DigitalProduct) {
    DigitalProduct d = (DigitalProduct) a;
    System.out.println(d.getDownloadUrl());
}
```

Modern Java (16+) does the test and the cast in one step, called **pattern matching**:

```java
if (a instanceof DigitalProduct d) {
    System.out.println(d.getDownloadUrl());
}
```

`d` is created only if the test is true, already of type `DigitalProduct`.

### 8.3 When is `instanceof` acceptable? (Lecture 5's casting trap)

```java
for (Product p : products) {
    // p.applyDiscount(0.10);     // does not compile: Product has no applyDiscount

    if (p instanceof Discountable d) {
        d.applyDiscount(0.10);
    }
}
```

Tutorial 5's experiment: the commented line fails with *cannot find symbol*, even though every object in the list really is discountable. **The reference type is `Product`, so only `Product`'s methods are visible.**

Lecture 5's distinction:

| Acceptable | Not acceptable |
| :-- | :-- |
| Asking about a **capability** (an interface): `instanceof Discountable` | Switching on **concrete classes** to decide behaviour: `instanceof DigitalProduct ... else if instanceof PhysicalProduct` |

And better still: if every item must be discountable, type the list that way (`List<Discountable>`) and no check is needed.

> **Exam rule.** If you find yourself writing an `instanceof` chain over subclasses to decide what to print or calculate, stop. Put an (abstract) method in the parent and override it in each child.

---

## 9. UML for abstraction

| Element | Notation |
| :-- | :-- |
| Abstract class | Class name in *italics* (handwritten: write `{abstract}` under the name) |
| Abstract method | Method in *italics* |
| Interface | `«interface»` written above the name (called a **stereotype**) |
| `extends` (generalisation) | **Solid** line, hollow triangle at the parent |
| `implements` (realisation) | **Dashed** line, hollow triangle at the interface |

```
            +---------------------------+          +---------------------------------+
            |        *Product*          |          |          «interface»            |
            +---------------------------+          |          Discountable           |
            | - id : Long               |          +---------------------------------+
            | - name : String           |          | + applyDiscount(rate : double)  |
            | - price : double          |          |       : void                    |
            +---------------------------+          | + getDiscountedPrice() : double |
            | + getName() : String      |          +---------------------------------+
            | + *getSummary() : String* |                   /_\       /_\
            +---------------------------+                    :         :
                        /_\                                  :         :   (dashed = implements)
                         |  (solid = extends)                :         :
          +--------------+---------------+                   :         :
          |                              |                   :         :
+-------------------------+   +--------------------------+   :         :
|     DigitalProduct      |   |     PhysicalProduct      |   :         :
+-------------------------+   +--------------------------+   :         :
| - downloadUrl : String  |...| - weightKg : double      |...:.........:
| - fileSizeMB : double   |   | - dimensions : String    |
+-------------------------+   +--------------------------+
| + getSummary() : String |   | + getSummary() : String  |
| + applyDiscount(...)    |   | + applyDiscount(...)     |
+-------------------------+   +--------------------------+
```

(`*asterisks*` stand for italics in plain text.) Overridden methods **are** listed again in the child box, because the child provides its own version. Inherited attributes are **not** repeated.

---

## 10. The mock-exam shape, fully written

An abstract parent with an abstract summary method, and two subclasses that each answer differently. This is the pattern for Tasks 1 and 2.

```java
public abstract class Course {

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

    public abstract String getSummary();
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

    @Override
    public String getSummary() {
        return "Online: " + getTitle() + " on " + platform + " (£" + getFee() + ")";
    }
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
    public void setRoom(String room) { this.room = room; }
    public void setCapacity(int capacity) { this.capacity = capacity; }

    @Override
    public String getSummary() {
        return "In person: " + getTitle() + " in " + room + " (" + capacity + " seats)";
    }
}
```

```java
List<Course> courses = new ArrayList<>();
courses.add(new OnlineCourse(1L, "Java Basics", 120.0, "Zoom"));
courses.add(new InPersonCourse(2L, "OOP Lab", 200.0, "C1.04", 30));

for (Course c : courses) {
    System.out.println(c.getSummary());
}
```

```
Online: Java Basics on Zoom (£120.0)
In person: OOP Lab in C1.04 (30 seats)
```

**The subclass checklist** (tick every item in the exam):

- [ ] `public class X extends Parent {`
- [ ] New fields only, `private`
- [ ] `public X() {}`
- [ ] Full constructor: parent's values first, then the new ones
- [ ] First line: `super(...)` with the parent's values, in the parent's order
- [ ] `this.newField = newField;`
- [ ] Getters (and setters) for the new fields
- [ ] Every abstract method overridden, each with `@Override`, exact name and parameters
- [ ] Parent's private fields reached through getters
- [ ] The summary text actually differs from the other subclass

---

## 11. Common mistakes

| Mistake | What happens | Fix |
| :-- | :-- | :-- |
| Misspelled override without `@Override` | Compiles; the parent's version runs. Silent. | Always write `@Override`. |
| Override with different parameters | It is an overload; the parent's version still runs. | Match the parameter list exactly. |
| Using the parent's `private` field in the override | "has private access". | Use the getter. |
| Making the override `private` or default | "attempting to assign weaker access privileges". | Keep it `public`. |
| `new` on an abstract class | "is abstract; cannot be instantiated". | Create a concrete subclass instead. |
| Abstract method with `{ }` | "abstract methods cannot have a body". | End with `;`. |
| Concrete child missing an abstract method | "is not abstract and does not override abstract method". | Write it, with `@Override`. |
| Calling a child method through a parent variable | "cannot find symbol". | Put the method in the parent (abstract), or use `instanceof` for a capability. |
| Downcasting without checking | `ClassCastException` at runtime. | `if (x instanceof Y y)`. |
| `instanceof` chain over subclasses | Works, but breaks Open/Closed and loses design marks. | Override instead. |
| UML: solid line for `implements` | Wrong notation. | Dashed line, hollow triangle. |

---

## 12. Practice

### 12.1 Tutorial 5 warm-up

Here `Product` is concrete (not yet abstract), with `getSummary()`. `DigitalProduct` overrides `getSummary()` and adds `getDownloadUrl()`.

```java
Product a = new Product(1L, "Mug", 8.50);
Product b = new DigitalProduct(2L, "E-book", 9.99, "/dl/2", 4.2);
DigitalProduct c = new DigitalProduct(3L, "Audiobook", 14.99, "/dl/3", 92.0);
```

Rule for column 2: look at the type on the **left** of `=`. Rule for column 3: look at what was built on the **right**.

| # | Statement | Compiles? | Which version runs, or why not? |
| :-- | :-- | :-- | :-- |
| 1 | `a.getSummary()` | | |
| 2 | `b.getSummary()` | | |
| 3 | `c.getSummary()` | | |
| 4 | `b.getDownloadUrl()` | | |
| 5 | `c.getDownloadUrl()` | | |
| 6 | `a.getDownloadUrl()` | | |
| 7 | `((DigitalProduct) b).getDownloadUrl()` | | |
| 8 | `((DigitalProduct) a).getDownloadUrl()` | | |

**Answers**

| # | Compiles? | Result |
| :-- | :-- | :-- |
| 1 | Yes | `Product`'s version. |
| 2 | Yes | `DigitalProduct`'s version: the object decides. |
| 3 | Yes | `DigitalProduct`'s version. |
| 4 | **No** | `Product` (the reference type) has no `getDownloadUrl()`. |
| 5 | Yes | `DigitalProduct`'s method. |
| 6 | **No** | `Product` has no `getDownloadUrl()`. |
| 7 | Yes | Runs fine: `b` really is a `DigitalProduct`. |
| 8 | Yes | **Runtime `ClassCastException`**: `a` is a plain `Product`. |

Rows 7 and 8 look identical to the compiler. Only the real object differs, and only the JVM, at runtime, can tell.

### 12.2 Quick questions

| # | Question | Answer |
| :-- | :-- | :-- |
| 1 | What is polymorphism? | One method call producing different behaviour depending on the real object's class. |
| 2 | What decides which overridden version runs? | The object type (right of `=`), at runtime. |
| 3 | What decides which methods you may call? | The reference type (left of `=`), at compile time. |
| 4 | Why write `@Override`? | So a misspelled or mismatched override is a compile error instead of a silent bug. |
| 5 | Can an abstract class have a constructor? | Yes. Children call it with `super(...)`. |
| 6 | Can a class implement two interfaces? | Yes. It can extend only one class. |
| 7 | Abstract class or interface for "can be exported to PDF", shared by invoices and reports? | Interface: a capability shared by unrelated classes. |
| 8 | Name the four pillars. | Encapsulation, inheritance, polymorphism, abstraction. |

### 12.3 Build it: `Shape`

Write:

- an abstract class `Shape` with a private `name` (`String`), a constructor, `getName()`, and abstract methods `double getArea()` and `String describe()`;
- `Circle extends Shape` with `radius`;
- `Rectangle extends Shape` with `width` and `height`;
- an interface `Scalable` with `void scale(double factor)`, implemented by both;
- in `main`, a `List<Shape>`: print each `describe()`, scale every shape by 2 using the interface, and print the total area.

Try for 25 minutes.

```java
public abstract class Shape {

    private String name;

    public Shape(String name) {
        this.name = name;
    }

    public String getName() { return name; }

    public abstract double getArea();

    public abstract String describe();
}
```

```java
public interface Scalable {
    void scale(double factor);
}
```

```java
public class Circle extends Shape implements Scalable {

    private double radius;

    public Circle(String name, double radius) {
        super(name);
        this.radius = radius;
    }

    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }

    @Override
    public String describe() {
        return "Circle " + getName() + " with radius " + radius;
    }

    @Override
    public void scale(double factor) {
        radius = radius * factor;
    }
}
```

```java
public class Rectangle extends Shape implements Scalable {

    private double width;
    private double height;

    public Rectangle(String name, double width, double height) {
        super(name);
        this.width = width;
        this.height = height;
    }

    @Override
    public double getArea() {
        return width * height;
    }

    @Override
    public String describe() {
        return "Rectangle " + getName() + " " + width + " x " + height;
    }

    @Override
    public void scale(double factor) {
        width = width * factor;
        height = height * factor;
    }
}
```

```java
List<Shape> shapes = new ArrayList<>();
shapes.add(new Circle("c1", 1.0));
shapes.add(new Rectangle("r1", 2.0, 3.0));

for (Shape s : shapes) {
    System.out.println(s.describe());
}

for (Shape s : shapes) {
    if (s instanceof Scalable sc) {
        sc.scale(2);
    }
}

double total = 0;
for (Shape s : shapes) {
    total += s.getArea();
}
System.out.printf("Total area: %.2f%n", total);
```

```
Circle c1 with radius 1.0
Rectangle r1 2.0 x 3.0
Total area: 36.57
```

(Check: after scaling, circle area = π × 2² ≈ 12.57, rectangle = 4 × 6 = 24, total ≈ 36.57.)

---

## 13. Glossary

| Term | Plain-English meaning |
| :-- | :-- |
| **Abstract class** | A class that cannot be created with `new`; a parent that may contain abstract methods. |
| **Abstract method** | A method header with no body, ending in `;`. Every concrete child must write it. |
| **Abstraction** | Showing what something does while hiding how; forbidding meaningless objects. |
| **Annotation** | A label beginning with `@` that gives the compiler or a framework extra information. |
| **Cast** | Telling Java to treat a reference as a more specific type: `(DigitalProduct) p`. |
| **`ClassCastException`** | Runtime error when a cast is wrong. |
| **Concrete class** | A class that is not abstract. |
| **Contract** | The set of methods an interface requires. |
| **`default` method** | An interface method that has a body. |
| **Downcast / upcast** | Parent → child (explicit, risky) / child → parent (automatic, safe). |
| **`implements`** | Signs an interface's contract. A class may implement many. |
| **`instanceof`** | Tests whether an object is of a given type. |
| **Interface** | A named list of methods a class promises to provide. |
| **Object type** | The class actually created with `new`. Decides what runs. |
| **Open/Closed Principle** | Open for extension, closed for modification. |
| **Overloading** | Same name, different parameters. Chosen at compile time. |
| **`@Override`** | Annotation asking the compiler to check that you really are overriding. |
| **Overriding** | A child redefining an inherited method with the same signature. |
| **Pattern matching** | `if (x instanceof Y y)`: test and cast in one step. |
| **Polymorphism** | One call, many behaviours, chosen by the object. |
| **Realisation** | UML's word for `implements` (dashed line, hollow triangle). |
| **Reference type** | The declared type of a variable. Decides what you may call. |
| **Runtime dispatch** | Choosing which overridden method to run from the real object, while the program runs. |
| **Stereotype** | A UML label in «guillemets», such as «interface». |
| **`super.method()`** | Calls the parent's version of a method. |
| **`toString()`** | Inherited from `Object`; called when an object is printed. Commonly overridden. |

---

## 14. One-page cheat sheet

```java
public abstract class Parent {                       // cannot be created with new
    private String name;
    public Parent() {}
    public Parent(String name) { this.name = name; }
    public String getName() { return name; }
    public abstract String getSummary();             // no body, ends with ;
}

public interface Discountable {                      // a capability contract
    void applyDiscount(double rate);                 // automatically public + abstract
}

public class Child extends Parent implements Discountable {   // one extends, many implements
    private double extra;

    public Child() {}
    public Child(String name, double extra) {
        super(name);
        this.extra = extra;
    }

    @Override                                        // ALWAYS write it
    public String getSummary() {                     // exact same header as parent
        return "Child: " + getName() + " " + extra;  // parent's private field: getter
    }

    @Override
    public void applyDiscount(double rate) { extra = extra * (1 - rate); }

    @Override
    public String toString() { return getSummary(); }
}

// using them
Parent p = new Child("A", 5);          // reference type Parent, object type Child
p.getSummary();                        // Child's version runs
// p.applyDiscount(0.1);               // compile error: Parent has no such method
if (p instanceof Discountable d) {     // ask about a capability
    d.applyDiscount(0.1);
}
list.stream().map(Parent::getSummary).toList();   // one polymorphic call, many answers
```

| Remember | |
| :-- | :-- |
| Left of `=` decides what you can call | Right of `=` decides what runs |
| Override = same name and parameters | Overload = different parameters |
| `@Override` on every override | Typos otherwise fail silently |
| Abstract class: state + shared code + forced methods | Interface: capability, many per class |
| Abstract method ends with `;` | Concrete children must implement it |
| `instanceof` for capabilities, not for types | Never an `instanceof` chain over subclasses |
| UML: italics = abstract | «interface»; dashed line + hollow triangle = implements |

---

## 15. Am I ready for Phase 6?

- [ ] I can explain what is wrong with an `instanceof` chain over subclasses. (§1)
- [ ] I can override a method correctly with `@Override`, and explain what `@Override` protects against. (§2)
- [ ] I can tell overriding from overloading. (§2.5)
- [ ] I can predict which version runs given the reference type and object type. (§3)
- [ ] I can make a class abstract with an abstract method, and explain the two compile errors that follow. (§4)
- [ ] I can write an interface and implement it differently in two classes. (§5)
- [ ] I can choose between an abstract class and an interface, with a reason. (§6)
- [ ] I can name the four pillars and point to code for each. (§7)
- [ ] I can use `instanceof` pattern matching, and explain `ClassCastException`. (§8)
- [ ] I can draw abstract classes and interfaces in UML. (§9)
- [ ] I can write both mock-exam subclasses from the checklist without looking. (§10)
- [ ] I built `Shape`, `Circle`, `Rectangle` and `Scalable` without copying. (§12.3)

### The question that opens Phase 6

The service can find a product by id, returning an `Optional`. In Lecture 3 the controller turned an empty result into `null` with `.orElse(null)`, so asking for product 999 returned an empty "200 OK", as if everything were fine. Changing it to `.orElseThrow()` instead crashed with a `NoSuchElementException` and a "500 Internal Server Error" that leaked Java internals to the user.

Neither is right. **How should a program fail on purpose: clearly, safely, and with a meaningful message?**

**Next: Phase 6, Exceptions.**
