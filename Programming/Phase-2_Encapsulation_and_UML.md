# Phase 2: Encapsulation and UML Class Diagrams

**Module:** 5COSC019W Object Oriented Programming
**Before this:** Phase 1 (classes, objects, constructors, `this`, references).
**Goal:** by the end you can write a properly protected class (private fields, getters, setters, validation, two constructors), link one class to another, and draw and read UML class diagrams.
**Time needed:** about 2.5 hours.

**Built from your files:** Lecture 2 (Encapsulation & UML Class Diagrams), Tutorial 2 (Encapsulation, the JSON Connection & UML), and the "Next week" slide of the Pure OOP Week 01 lecture.

> **Why this phase matters for the exam.** Every exam task uses classes written this way. Tasks 1 and 2 ask you to write fields and constructors in exactly this style, and Task 3 (validation) builds on the setter checks in Section 6.

---

## Where we are on the roadmap

| Phase | Title | Status |
| :-- | :-- | :-- |
| Pre-Phase 0 | Absolute Java Basics | Done |
| Phase 1 | Classes and Objects | Done |
| **Phase 2** | **Encapsulation and UML Class Diagrams** | **You are here** |
| Phase 3 | Collections and Generics | Next |
| Phase 4 | Inheritance | |
| Phase 5 | Polymorphism and Abstraction | |
| Phase 6 | Exceptions | |
| Phase 7 | Dependencies and Layered Design | |
| Phase 8 | Exam Consolidation | |

---

## 1. The problem: anyone can break the object

Phase 1 ended with this code, which Java currently allows:

```java
EquipmentItem laptop = new EquipmentItem(101, "Dell Laptop", 185000.00, true);

laptop.replacementCost = -5000;
laptop.available = false;
```

Any code, anywhere in the program, can reach inside the object and write nonsense into it. A negative price. An empty name. The object has no way to defend itself.

- **Analogy: a bank account.** Imagine if anyone could walk up to the bank's computer and type a new number into your balance. Banks do not work that way. You can only change your balance through approved actions, such as "deposit" and "withdraw", and each action checks the rules (you cannot withdraw more than you have).

**Encapsulation** is the bank's approach applied to objects:

1. **Hide the data.** Nobody outside the class can touch the fields directly.
2. **Offer approved actions.** Outside code goes through methods, and the methods enforce the rules.

The word comes from "capsule": the data is sealed inside, and the only way in or out is through the openings you choose.

Encapsulation is the first of the **four pillars of OOP** your module names (encapsulation, inheritance, polymorphism, abstraction). Phases 4 and 5 cover the other three.

---

## 2. Access modifiers: who is allowed to see what

### 2.1 The four levels

An **access modifier** is a keyword placed in front of a field, method, constructor or class that says **who is allowed to use it**. Java has four levels.

| Modifier | UML symbol | Who can use it | Analogy |
| :-- | :-- | :-- | :-- |
| `private` | `-` | **Only code inside the same class** | Your diary: only you |
| *(nothing written)*, called **default** or **package-private** | `~` | Code in the same **package** | Your family's fridge: anyone in the house |
| `protected` | `#` | Same package, **plus** child classes anywhere (Phase 4) | Family heirlooms: the household, plus your children who moved away |
| `public` | `+` | **Everyone, everywhere** | A public park |

A **package** is a folder of related classes. In the Spring project it is `uk.ac.westminster.productsapi`. The first line of each file in that folder says so:

```java
package uk.ac.westminster.productsapi;
```

### 2.2 The rule of thumb (from Lecture 2)

> **Fields should almost always be `private`. Methods that are part of the class's public face should be `public`.**

Remember the slogan from the lecture's key takeaway: **private data, public behaviour.**

### 2.3 What `private` does in practice

```java
public class Product {

    private Long id;
    private String name;
    private double price;

    public Product(Long id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }
}
```

In another class:

```java
Product p = new Product(1L, "Laptop", 999.99);
System.out.println(p.name);     // COMPILE ERROR
```

```
error: name has private access in Product
```

| Piece | What it means |
| :-- | :-- |
| `private Long id;` | Only code written inside `Product` may read or change `id`. |
| `p.name` from another class | Refused. The compiler stops you before the program runs. |

Lecture 2 asks: *"Why does the compiler stop us here, and is that a good thing?"* Yes, it is good. Java is protecting the object's internal state. This is encapsulation **enforced by the compiler**, not just a polite convention.

Notice the constructor is still allowed to write `this.name = name;`. The constructor is inside the class, so `private` does not block it.

---

## 3. Getters: controlled reading

The fields are now locked. But outside code still needs to **read** the name and price. The answer is a small public method whose only job is to hand the value back.

A **getter** (also called an **accessor**) is a public method that returns the value of one private field.

```java
public Long getId() {
    return id;
}

public String getName() {
    return name;
}

public double getPrice() {
    return price;
}
```

`public String getName() { return name; }`

| Piece | What it means |
| :-- | :-- |
| `public` | Anyone may call it. |
| `String` | Returns text: the same type as the `name` field. |
| `getName` | `get` + the field name with a **capital first letter**. |
| `()` | No parameters. It only reads. |
| `return name;` | Hand back the value of this object's `name` field. |

Using it:

```java
Product p = new Product(1L, "Laptop", 999.99);
System.out.println(p.getName());    // Laptop
System.out.println(p.getPrice());   // 999.99
```

You will often see a getter written on one line. It is identical; Java ignores line breaks:

```java
public String getName() { return name; }
```

- **Analogy: a museum display case.** You can look at the exhibit through the glass (the getter), but you cannot reach in and take it.

---

## 4. Setters: controlled changing

A **setter** (also called a **mutator**) is a public method that changes one private field.

```java
public void setPrice(double price) {
    this.price = price;
}
```

| Piece | What it means |
| :-- | :-- |
| `public` | Anyone may call it. |
| `void` | Returns nothing. Its job is to change, not to answer. |
| `setPrice` | `set` + the field name with a capital first letter. |
| `(double price)` | The new value, handed in. Same type as the field. |
| `this.price = price;` | Store it in the field. Same pattern as the constructor. |

Using it:

```java
p.setPrice(899.99);
System.out.println(p.getPrice());   // 899.99
```

"Wait," you might think, "a setter that stores anything it is given is no safer than a public field." Correct, and that is exactly why Section 6 adds checks inside setters. A setter is the **place** where the rules live. A public field has nowhere for rules to go.

---

## 5. The JavaBean naming convention

A **convention** is a habit that everyone agrees to follow. A **JavaBean** is a class that follows this particular naming habit: private fields, a no-argument constructor, and getters and setters named in a fixed pattern. Tools (including Spring) rely on these exact names.

| Field | Getter | Setter |
| :-- | :-- | :-- |
| `private String name;` | `public String getName()` | `public void setName(String name)` |
| `private double price;` | `public double getPrice()` | `public void setPrice(double price)` |
| `private Long id;` | `public Long getId()` | `public void setId(Long id)` |
| `private boolean inStock;` | `public boolean isInStock()` | `public void setInStock(boolean inStock)` |

**Rules:**

1. `get` or `set`, then the field name with its **first letter capitalised**: `name` → `getName`.
2. For a `boolean` field, the getter uses **`is`** instead of `get`: `inStock` → `isInStock()`.
3. The getter's return type matches the field's type **exactly**: `getPrice()` returns `double`, not `Double`.

`getname()`, `GetName()` and `get_name()` do **not** count. Tutorial 2's troubleshooting section names this as a common cause of missing data.

IntelliJ can write getters and setters for you: put the cursor inside the class, press **Alt + Insert** → *Getter and Setter* → select the fields. Learn to type them by hand first; in the exam, the shortcut saves time.

---

## 6. Validation: the real reason for setters

### 6.1 Rejecting bad values

This is the answer to the Phase 1 question "how can an object protect its own state?"

```java
public void setReplacementCost(double replacementCost) {
    if (replacementCost < 0) {
        throw new IllegalArgumentException("Replacement cost cannot be negative");
    }
    this.replacementCost = replacementCost;
}
```

| Line | What it means |
| :-- | :-- |
| `if (replacementCost < 0) {` | Check the new value **before** storing it. The same validate-first pattern as Pre-Phase 0. |
| `throw new IllegalArgumentException("...");` | Refuse the value and stop immediately. Explained below. |
| `this.replacementCost = replacementCost;` | Only reached if the check passed. |

**`throw` in one paragraph (Phase 6 covers it fully).** `throw` means "stop what you are doing right now and report a problem". `IllegalArgumentException` is a ready-made Java class meaning "you handed me an unacceptable value". `new` builds one, with a message explaining what went wrong. When it is thrown, the method ends at once, so the bad value is never stored. If nothing deals with the problem, the program stops and prints the message.

```java
laptop.setReplacementCost(-5000);
```

```
Exception in thread "main" java.lang.IllegalArgumentException: Replacement cost cannot be negative
```

The object refused. It is still in a valid state.

- **Analogy: a bouncer at the door.** The setter is the bouncer. Valid values get in. Invalid ones are turned away with a reason.

### 6.2 Validating in the constructor too

If only the setter checks, someone can still write `new EquipmentItem(101, "Laptop", -5000, true)`. The constructor is a second door. Guard it too.

The cleanest way: **let the constructor call the setter**, so the rule is written once.

```java
public EquipmentItem(long id, String name, double replacementCost, boolean available) {
    this.id = id;
    setName(name);
    setReplacementCost(replacementCost);
    this.available = available;
}
```

Now both doors run the same check. Writing a rule once and using it everywhere is a principle you will hear often: **DRY**, "Don't Repeat Yourself".

### 6.3 Checking text

For a `String`, the usual checks are "is it missing?" and "is it empty?":

```java
public void setName(String name) {
    if (name == null || name.isBlank()) {
        throw new IllegalArgumentException("Name is required");
    }
    this.name = name;
}
```

| Piece | What it means |
| :-- | :-- |
| `name == null` | No text object at all. (`==` is right here, because we are asking "is there an object?", not comparing letters.) |
| `\|\|` | OR. |
| `name.isBlank()` | A built-in `String` method: `true` if the text is empty or only spaces. |
| Order matters | `null` is checked **first**. If `name` is `null`, Java stops at the `\|\|` (the answer is already true) and never calls `isBlank()`, which would crash with `NullPointerException`. This stopping early is called **short-circuiting**. |

### 6.4 Behaviour methods instead of setters

Sometimes a plain setter is the wrong tool. For `available`, the real-world actions are "borrow" and "return", not "set available to some value". Methods named after real actions are clearer and can enforce rules:

```java
public void borrow() {
    if (!available) {
        throw new IllegalStateException("Item is already on loan");
    }
    available = false;
}

public void returnItem() {
    available = true;
}
```

| Piece | What it means |
| :-- | :-- |
| `!available` | NOT available. |
| `IllegalStateException` | Another ready-made Java problem type, meaning "this action is not allowed in the object's current state". Different from `IllegalArgumentException` (bad value handed in). |
| `available = false;` | Only reached if the item was available. |

These are **commands** (Phase 1, Section 7.6): methods that change state, each guarding its own rule.

---

## 7. To setter, or not to setter? Immutability

### 7.1 The idea

Lecture 2's advice:

| Situation | Decision | Example |
| :-- | :-- | :-- |
| The field genuinely needs to change after the object is created | Provide a setter | A shopping cart's `quantity` |
| The value is fixed for the object's whole life | **No setter** | A product's `id` |

**Fewer setters means fewer ways for the object to get into a bad state.** An object whose fields can never change after construction is called **immutable** ("not changeable"). The opposite is **mutable**.

Lecture 2's quick check: *"If a field has a getter but no setter, can it still change value?"* **Yes, but only from inside the class.** Outside code cannot change it.

### 7.2 The `final` keyword

`final` on a field means **it must be given a value exactly once (in its declaration or in every constructor), and can never be changed after that**. The compiler enforces it.

```java
public class Product {

    private final Long id;
    private final String name;
    private final double price;

    public Product(Long id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public Long getId() { return id; }
    public String getName() { return name; }
    public double getPrice() { return price; }
}
```

| Piece | What it means |
| :-- | :-- |
| `private final double price;` | Private (hidden) and final (set once). |
| No `setPrice` | No way to change it from outside. |
| Any attempt to write `price = 5;` in a method | Compile error: "cannot assign a value to final variable price". |

> **A warning about the Lecture 2 slide.** The "Building an Immutable Product" slide shows `final` fields **together with** an empty `public Product() {}` constructor. That combination does **not** compile: every constructor must give every `final` field a value, and the empty one gives them nothing. The error is `variable id might not have been initialized`. Tutorial 2's code (no `final`, both constructors) is the version that works. If you see `final` fields in exam code, do not add an empty constructor to that class.

---

## 8. Two constructors: overloading

### 8.1 More than one constructor is allowed

From Tutorial 2 (Code Reference 3):

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
}
```

| Constructor | Name | Purpose (from Lecture 2) |
| :-- | :-- | :-- |
| `public Product() {}` | **No-argument constructor** | Exists so frameworks can build an empty object first and fill it in afterwards. |
| `public Product(Long id, String name, double price)` | **Full constructor** | What you call in your own code. Guarantees every field is set at creation. |

`public Product() {}` piece by piece: `public` (anyone may use it), `Product` (class name, so it is a constructor), `()` (no parameters), `{}` (an empty body: do nothing extra; the fields keep their defaults).

Having several constructors (or methods) with **the same name but different parameter lists** is called **overloading**. Java decides which one to run by looking at the arguments:

```java
Product a = new Product();                          // runs the no-arg one
Product b = new Product(1L, "Laptop", 999.99);      // runs the full one
```

Two overloaded versions must differ in the **number, types or order** of parameters. Two constructors both taking `(Long, String, double)` is a compile error.

Tutorial 2: "Two constructors look like an error: they are not."

### 8.2 Why does Spring need the empty one?

You will study this properly in the Spring chat. In short, when JSON data arrives in a web request (for example `{ "name": "Mouse", "price": 19.99 }`), a library called **Jackson** turns it into a Java object in two steps: it calls `new Product()` with no arguments, then fills each field in. Without a no-arg constructor, step one fails and the request is rejected with an HTTP 400 error.

Lecture 2 quick check, Q1: *"Why does `Product` need a no-arg constructor if it already has a full one?"* **Jackson uses it to construct the object before setting fields during deserialisation.** ("Deserialisation" means turning text, such as JSON, back into an object.)

### 8.3 One constructor calling another: `this(...)`

Not in your slides, but useful in the exam. A constructor can hand its work to another constructor of the same class with `this(...)`:

```java
public Product(String name, double price) {
    this(null, name, price);
}
```

This means "run the full constructor with these values". `this(...)` must be the **first line** of the constructor. (`this(...)` with brackets is a constructor call; `this.name` with a dot is a field. Same word, two jobs.)

---

## 9. The Spring connection: getters decide what becomes JSON

This is Lecture 2's live demonstration, explained just enough to understand the design rule. The Spring chat covers the Spring code.

**JSON** (JavaScript Object Notation) is a plain-text way to write down structured data so it can travel over the internet:

```json
{
  "id": 1,
  "name": "Laptop",
  "price": 999.99
}
```

`{ }` wraps one object; each `"key": value` pair is one field; text is in quotes; numbers are not.

An object in memory cannot travel down a network cable. When a Spring endpoint returns a `Product`, the Jackson library writes it out as JSON text. This is called **serialisation** (object → text).

**The crucial fact: Jackson does not read private fields. It calls the public getters.** For each `getX()` it finds, it writes a field `"x"`.

| `Product` version | JSON returned | Why |
| :-- | :-- | :-- |
| Public fields | `{"id":1,"name":"Laptop","price":999.99}` | Jackson can see public fields. |
| Private fields, **no getters** | `{}` | Nothing for Jackson to call. Status 200. **No error anywhere.** |
| Private fields, all three getters | `{"id":1,"name":"Laptop","price":999.99}` | Each getter produces one field. |
| Private fields, `getName()` removed | `{"id":1,"price":999.99}` | `name` silently vanishes. |
| Getter misspelled `getname()` | `name` is missing | Jackson only recognises the exact JavaBean name. |

Tutorial 2's key point: *no compile error, no exception, no warning, a 200 OK with part of the data missing.* This is the most common way a Spring API quietly returns the wrong thing.

> **Exam rule.** Every private field you want visible in the API needs a correctly named public getter. When Section B of your exam checks a response and a field is missing, the cause is almost always a missing or misspelled getter.

---

## 10. One class inside another: association (HAS-A)

### 10.1 A field whose type is another class

A field's type does not have to be `int` or `String`. It can be any class, including one you wrote. Tutorial 2 builds a customer who has an address.

**`Address.java`**

```java
public class Address {

    private String street;
    private String city;
    private String postcode;

    public Address() {}

    public Address(String street, String city, String postcode) {
        this.street = street;
        this.city = city;
        this.postcode = postcode;
    }

    public String getStreet() { return street; }
    public String getCity() { return city; }
    public String getPostcode() { return postcode; }
}
```

**`Customer.java`**

```java
public class Customer {

    private Long id;
    private String name;
    private String email;
    private Address address;

    public Customer() {}

    public Customer(Long id, String name, String email, Address address) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.address = address;
    }

    public Long getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public Address getAddress() { return address; }
}
```

The new line: `private Address address;`

| Piece | What it means |
| :-- | :-- |
| `private` | Hidden, like every field. |
| `Address` | The type is **your own class**. |
| `address` | The field name. Lower case, same word: a common habit. |

Because `Address` is a class, this field holds a **reference** to an `Address` object (Phase 1, Section 10), not the address data itself.

### 10.2 Building them

```java
Address address = new Address("115 New Cavendish Street", "London", "W1W 6UW");
Customer ada = new Customer(1L, "Ada Lovelace", "ada@example.com", address);

System.out.println(ada.getName());                  // Ada Lovelace
System.out.println(ada.getAddress().getCity());     // London
```

| Step | What happens |
| :-- | :-- |
| 1 | Build the `Address` object first. |
| 2 | Pass its reference into the `Customer` constructor, which stores it in the `address` field. |
| 3 | `ada.getAddress()` returns the `Address` object. |
| 4 | `.getCity()` is then called **on that returned object**. |

Read `ada.getAddress().getCity()` left to right: "ada's address's city". Calling one method on the result of another like this is called **method chaining**.

```
   Customer object                       Address object
  +-----------------------+            +------------------------------+
  | id = 1                |            | street = "115 New Cav..."    |
  | name = "Ada Lovelace" |            | city = "London"              |
  | email = "ada@..."     |            | postcode = "W1W 6UW"         |
  | address  ------------------------> |                              |
  +-----------------------+            +------------------------------+
```

### 10.3 Association and HAS-A

When one class has a field whose type is another class, the two classes have an **association**: a lasting link between them. You describe it with the phrase **HAS-A**: "a `Customer` **has an** `Address`."

- **Analogy:** a person has a phone. The phone is a separate thing with its own features; the person simply holds onto it.

Phase 4 contrasts HAS-A with **IS-A** (inheritance). Getting those two right is a common exam and report question.

In JSON, the address appears as a **nested object** (an object inside an object, with its own braces):

```json
{
  "id": 1,
  "name": "Ada Lovelace",
  "email": "ada@example.com",
  "address": {
    "street": "115 New Cavendish Street",
    "city": "London",
    "postcode": "W1W 6UW"
  }
}
```

Remove `getAddress()` and the **whole** address block disappears at once.

---

## 11. `static` members: one shared copy

### 11.1 Instance fields versus static fields

Every field so far has been an **instance field**: each object gets its own copy. A **static field** is different: **there is exactly one copy, shared by the whole class**, no matter how many objects exist.

- **Analogy:** each student has their own name badge (instance field). The classroom has one clock on the wall that everybody shares (static field).

Lecture 2's example: counting how many products have been created.

```java
public class Product {

    private static int instanceCount = 0;

    private Long id;

    public Product() {
        instanceCount++;
    }

    public static int getInstanceCount() {
        return instanceCount;
    }
}
```

| Line | What it means |
| :-- | :-- |
| `private static int instanceCount = 0;` | One shared counter for the whole class, starting at 0. |
| `instanceCount++;` in the constructor | Every time any `Product` is built, the shared counter goes up by one. |
| `public static int getInstanceCount()` | A static method: belongs to the class, so it can be called without any object. |

```java
Product p1 = new Product();
Product p2 = new Product();
Product p3 = new Product();

System.out.println(Product.getInstanceCount());   // 3
```

Static members are called on the **class name**: `Product.getInstanceCount()`. You have used this since Pre-Phase 0: `Math.PI` and `System.out` are static members.

Lecture 2 notes that `p1.getInstanceCount()` also compiles, but is misleading style. Always use the class name.

A static method **cannot** use instance fields or `this`, for the same reason as `main` in Phase 1: there is no "current object".

### 11.2 Constants

`static` and `final` together make a **constant**: one shared value that never changes. By convention, constants are written in CAPITAL_LETTERS with underscores.

```java
public static final double LATE_FEE_PER_DAY = 2.50;
```

This is better than writing `2.50` inside the method: the number now has a name, and if the fee changes you edit one line.

---

## 12. UML class diagrams

### 12.1 What UML is

**UML** (Unified Modeling Language) is a standard way of **drawing** software designs, so that a whole team can agree on a design before anyone writes code. A **class diagram** is the UML drawing of classes and how they relate. Your coursework report is marked on this notation (learning outcome LO5).

Lecture 2's message: *design starts on paper, not in the IDE.* Mistakes found in a diagram are cheap to fix; mistakes found in finished code are not.

### 12.2 The class box

Every class is a box with **three compartments, always in this order**:

```
+------------------------------------------------+
|                    Product                     |   1. class name
+------------------------------------------------+
| - id : Long                                    |
| - name : String                                |   2. attributes (fields)
| - price : double                               |
+------------------------------------------------+
| + Product()                                    |
| + Product(id : Long, name : String,            |
|           price : double)                      |   3. operations (constructors and methods)
| + getId() : Long                               |
| + getName() : String                           |
| + getPrice() : double                          |
+------------------------------------------------+
```

### 12.3 The syntax rules

| Element | Format | Example | Java |
| :-- | :-- | :-- | :-- |
| Attribute | `visibility name : Type` | `- price : double` | `private double price;` |
| Method | `visibility name(param : Type) : ReturnType` | `+ setPrice(price : double) : void` | `public void setPrice(double price)` |
| Constructor | Like a method, **no return type** | `+ Product(id : Long, name : String)` | `public Product(Long id, String name)` |
| Static member | **Underlined** | <u>`- instanceCount : int`</u> | `private static int instanceCount;` |

**Visibility symbols:**

| Symbol | Meaning |
| :-- | :-- |
| `+` | public |
| `-` | private |
| `#` | protected |
| `~` | default (package) |

The biggest difference from Java: **in UML the type comes after the name, following a colon.** Java says `double price`; UML says `price : double`.

Lecture 2 quick check: *"How would you write `private double price` in UML?"* → `- price : double`.

### 12.4 Drawing an association

A **solid line** between two boxes, with an **open arrowhead** pointing at the class being held, reads "has a":

```
+-------------------------+                 +-------------------------+
|        Customer         |                 |         Address         |
+-------------------------+                 +-------------------------+
| - id : Long             |                 | - street : String       |
| - name : String         |---------------->| - city : String         |
| - email : String        |                 | - postcode : String     |
| - address : Address     |                 +-------------------------+
+-------------------------+                 | + getStreet() : String  |
| + getAddress() : Address|                 | + getCity() : String    |
+-------------------------+                 | + getPostcode() : String|
                                            +-------------------------+
         "a Customer has an Address"
```

The arrow points **at** `Address` (the one being held), not the other way.

### 12.5 The checklist markers use (Tutorial 2, Appendix B)

- [ ] Exactly three compartments: name, attributes, methods.
- [ ] Class name spelled exactly as in the Java file, capital first letter.
- [ ] Every attribute is `visibility name : Type`.
- [ ] Every private field marked `-`.
- [ ] Every public method marked `+`.
- [ ] Constructors listed with no return type.
- [ ] Getter return types match field types exactly (`getPrice() : double`, not `Double`).
- [ ] Associations: solid line, arrow pointing at the held class.
- [ ] Static members underlined.

### 12.6 Translating in both directions

**Diagram → code.** Given `- discountRate : double` and `+ applyDiscount(rate : double) : void`:

```java
private double discountRate;

public void applyDiscount(double rate) {
    this.discountRate = rate;
}
```

**Code → diagram.** Given:

```java
public class Book {
    private String isbn;
    protected int pages;
    String genre;
    public static int count;
    public boolean isLong(int limit) { return pages > limit; }
}
```

```
+------------------------------------+
|               Book                 |
+------------------------------------+
| - isbn : String                    |
| # pages : int                      |
| ~ genre : String                   |
| + count : int        (underlined)  |
+------------------------------------+
| + isLong(limit : int) : boolean    |
+------------------------------------+
```

---

## 13. The complete encapsulated `EquipmentItem`

Phase 1's class, rebuilt with everything in this phase. Type it out.

```java
public class EquipmentItem {

    public static final double LATE_FEE_PER_DAY = 2.50;
    private static int itemCount = 0;

    private final long id;
    private String name;
    private double replacementCost;
    private boolean available;

    public EquipmentItem(long id, String name, double replacementCost, boolean available) {
        this.id = id;
        setName(name);
        setReplacementCost(replacementCost);
        this.available = available;
        itemCount++;
    }

    public long getId() { return id; }
    public String getName() { return name; }
    public double getReplacementCost() { return replacementCost; }
    public boolean isAvailable() { return available; }

    public void setName(String name) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name is required");
        }
        this.name = name;
    }

    public void setReplacementCost(double replacementCost) {
        if (replacementCost < 0) {
            throw new IllegalArgumentException("Replacement cost cannot be negative");
        }
        this.replacementCost = replacementCost;
    }

    public void borrow() {
        if (!available) {
            throw new IllegalStateException("Item is already on loan");
        }
        available = false;
    }

    public void returnItem() {
        available = true;
    }

    public double calculateLateFee(int daysLate) {
        return daysLate * LATE_FEE_PER_DAY;
    }

    public boolean isExpensive(double threshold) {
        return replacementCost > threshold;
    }

    public static int getItemCount() {
        return itemCount;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        EquipmentItem laptop = new EquipmentItem(101, "Dell Laptop", 185000.00, true);
        EquipmentItem camera = new EquipmentItem(102, "Canon Camera", 120000.00, true);

        System.out.println(laptop.getName());
        System.out.println(laptop.isAvailable());

        laptop.borrow();
        System.out.println(laptop.isAvailable());

        System.out.println(camera.calculateLateFee(4));
        System.out.println(EquipmentItem.getItemCount());

        laptop.setReplacementCost(-5000);
    }
}
```

**Output:**

```
Dell Laptop
true
false
10.0
2
Exception in thread "main" java.lang.IllegalArgumentException: Replacement cost cannot be negative
```

Design decisions, and why:

| Decision | Reason |
| :-- | :-- |
| `id` is `final` with no setter | An item's id never changes. |
| `name` and `replacementCost` have validating setters | They may change, but only to sensible values. |
| The constructor calls the setters | The rules are written once and guard both doors. |
| `available` has no setter, only `borrow()` and `returnItem()` | Real actions, each with its own rule. |
| `isAvailable()` uses `is` | It is a `boolean` getter. |
| `LATE_FEE_PER_DAY` is a constant | A named number, changed in one place. |

The UML for it:

```
+-------------------------------------------------------------+
|                        EquipmentItem                        |
+-------------------------------------------------------------+
| + LATE_FEE_PER_DAY : double                    (underlined) |
| - itemCount : int                              (underlined) |
| - id : long                                                 |
| - name : String                                             |
| - replacementCost : double                                  |
| - available : boolean                                       |
+-------------------------------------------------------------+
| + EquipmentItem(id : long, name : String,                   |
|                 replacementCost : double, available : boolean)|
| + getId() : long                                            |
| + getName() : String                                        |
| + getReplacementCost() : double                             |
| + isAvailable() : boolean                                   |
| + setName(name : String) : void                             |
| + setReplacementCost(replacementCost : double) : void       |
| + borrow() : void                                           |
| + returnItem() : void                                       |
| + calculateLateFee(daysLate : int) : double                 |
| + isExpensive(threshold : double) : boolean                 |
| + getItemCount() : int                         (underlined) |
+-------------------------------------------------------------+
```

---

## 14. Common mistakes

| Mistake | What happens | Fix |
| :-- | :-- | :-- |
| Making fields `public` | Anyone can put the object in a bad state. | `private`, always. |
| Misspelled getter (`getname`) | Compiles. In Spring, the field silently disappears from the JSON. | `get` + capitalised field name. |
| `getAvailable()` for a boolean | Works in plain Java, but breaks the JavaBean convention. | `isAvailable()`. |
| Setter without `this.` (`price = price;`) | Silent: the field never changes. | `this.price = price;` |
| Validating in the setter but not the constructor | `new` can still create a bad object. | Constructor calls the setter. |
| Writing your own full constructor and forgetting the no-arg one | Plain Java is fine; in Spring, incoming JSON fails with 400. | Add `public X() {}` (unless fields are `final`). |
| `final` fields plus an empty constructor | Compile error: "variable might not have been initialized". | Drop one of them. |
| Accessing a private field from another class | Compile error: "has private access". | Use the getter. |
| UML type before the name (`- double price`) | Wrong notation. | `- price : double` |
| Association arrow pointing the wrong way | Wrong meaning. | Arrow points at the held class. |

---

## 15. Practice

### 15.1 Quick checks (from Lecture 2)

| # | Question | Model answer |
| :-- | :-- | :-- |
| 1 | Why does `Product` need a no-arg constructor if it already has a full one? | Jackson uses it to construct the object before setting fields during deserialisation. |
| 2 | If a field has a getter but no setter, can it still change value? | Yes, from inside the class. Outside code cannot change it directly. |
| 3 | What JSON would you expect from a private field with no getter? | The field is simply left out. No error, no warning. |
| 4 | What does an underlined attribute in UML mean? | A static member, shared by the class, not one per object. |
| 5 | How would you write `private double price` in UML? | `- price : double` |
| 6 | Why draw the diagram before writing code? | It surfaces design decisions and mistakes cheaply, before implementation effort is spent. |
| 7 | What is encapsulation? | Keeping an object's data private and allowing access only through methods that enforce its rules. |
| 8 | What is the difference between `IllegalArgumentException` and `IllegalStateException`? | The first means "the value you passed is unacceptable"; the second means "this action is not allowed in the object's current state". |

### 15.2 Spot the error

```java
// A
public class Student {
    private String name;
    public String getname() { return name; }
}

// B
public class Student {
    private boolean enrolled;
    public boolean getEnrolled() { return enrolled; }
}

// C  (in Main)
Student s = new Student("Kamal");
s.name = "Nimal";

// D
public class Student {
    private int age;
    public void setAge(int age) {
        age = age;
    }
}

// E
public class Student {
    private final String id;
    public Student() {}
    public Student(String id) { this.id = id; }
}
```

| | Problem | Fix |
| :-- | :-- | :-- |
| A | `getname` breaks the JavaBean convention (lower-case n). | `getName()` |
| B | A boolean getter should start with `is`. | `isEnrolled()` |
| C | `name` is private: compile error "has private access". | `s.setName("Nimal");` (and write that setter) |
| D | Missing `this.`: the field is never set. | `this.age = age;` |
| E | The empty constructor never sets the `final` field `id`: compile error. | Remove `final` or remove the empty constructor. |

### 15.3 Build it: `BankAccount`

Write a `BankAccount` class with:

- private `accountNumber` (`String`, never changes) and `balance` (`double`);
- one constructor taking both, rejecting a negative starting balance;
- getters for both, no setters;
- `deposit(double amount)` that rejects zero or negative amounts;
- `withdraw(double amount)` that rejects zero or negative amounts, and rejects withdrawing more than the balance;
- a static count of accounts created, with a static getter.

Then draw its UML. Try it for 20 minutes before reading on.

```java
public class BankAccount {

    private static int accountCount = 0;

    private final String accountNumber;
    private double balance;

    public BankAccount(String accountNumber, double openingBalance) {
        if (openingBalance < 0) {
            throw new IllegalArgumentException("Opening balance cannot be negative");
        }
        this.accountNumber = accountNumber;
        this.balance = openingBalance;
        accountCount++;
    }

    public String getAccountNumber() { return accountNumber; }
    public double getBalance() { return balance; }

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit must be positive");
        }
        balance += amount;
    }

    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal must be positive");
        }
        if (amount > balance) {
            throw new IllegalStateException("Insufficient funds");
        }
        balance -= amount;
    }

    public static int getAccountCount() {
        return accountCount;
    }
}
```

```java
BankAccount acc = new BankAccount("ACC-001", 100.0);
acc.deposit(50.0);
acc.withdraw(30.0);
System.out.println(acc.getBalance());               // 120.0
System.out.println(BankAccount.getAccountCount());  // 1
acc.withdraw(500.0);   // IllegalStateException: Insufficient funds
```

```
+----------------------------------------------------------+
|                       BankAccount                        |
+----------------------------------------------------------+
| - accountCount : int                        (underlined) |
| - accountNumber : String                                 |
| - balance : double                                       |
+----------------------------------------------------------+
| + BankAccount(accountNumber : String,                    |
|               openingBalance : double)                   |
| + getAccountNumber() : String                            |
| + getBalance() : double                                  |
| + deposit(amount : double) : void                        |
| + withdraw(amount : double) : void                       |
| + getAccountCount() : int                   (underlined) |
+----------------------------------------------------------+
```

Notice there is no `setBalance`. The balance can only change through `deposit` and `withdraw`, which is exactly the bank analogy from Section 1.

### 15.4 Diagram to code

Write the Java for this diagram, then check below.

```
+------------------------------------------+
|                 Course                   |
+------------------------------------------+
| - code : String                          |
| - title : String                         |
| - credits : int                          |
+------------------------------------------+
| + Course()                               |
| + Course(code : String, title : String,  |
|          credits : int)                  |
| + getCode() : String                     |
| + getTitle() : String                    |
| + getCredits() : int                     |
| + setCredits(credits : int) : void       |
+------------------------------------------+
```

```java
public class Course {

    private String code;
    private String title;
    private int credits;

    public Course() {}

    public Course(String code, String title, int credits) {
        this.code = code;
        this.title = title;
        this.credits = credits;
    }

    public String getCode() { return code; }
    public String getTitle() { return title; }
    public int getCredits() { return credits; }

    public void setCredits(int credits) {
        this.credits = credits;
    }
}
```

---

## 16. Glossary

| Term | Plain-English meaning |
| :-- | :-- |
| **Access modifier** | A keyword (`private`, `protected`, `public`, or none) saying who may use something. |
| **Accessor** | Another word for getter. |
| **Association** | A lasting link between two classes, where one holds a reference to the other. |
| **Constant** | A `static final` value that never changes, written in CAPITALS. |
| **Default (package-private) access** | No modifier written: visible only inside the same package. |
| **Deserialisation** | Turning text (such as JSON) back into an object. |
| **DRY** | "Don't Repeat Yourself": write each rule once. |
| **Encapsulation** | Hiding an object's data and allowing access only through methods that enforce its rules. |
| **`final`** | Assigned once, never changed. |
| **Getter** | A public method that returns the value of a private field. |
| **HAS-A** | The phrase describing an association: "a Customer has an Address". |
| **`IllegalArgumentException`** | Java's ready-made "the value you passed is unacceptable" problem. |
| **`IllegalStateException`** | Java's ready-made "not allowed in the current state" problem. |
| **Immutable** | Cannot be changed after creation. |
| **Instance field** | A field each object has its own copy of. |
| **Jackson** | The library Spring uses to convert objects to and from JSON. It reads getters. |
| **JavaBean** | A class following the private-fields, no-arg-constructor, getX/setX/isX convention. |
| **JSON** | A plain-text format for structured data sent over the web. |
| **Mutable** | Can be changed. |
| **Mutator** | Another word for setter. |
| **Nested object** | An object inside another object, in JSON or in Java. |
| **No-argument constructor** | A constructor with no parameters. |
| **Overloading** | Several constructors or methods with the same name but different parameter lists. |
| **Package** | A folder of related classes, declared by `package ...;` at the top of each file. |
| **`private`** | Visible only inside the same class. |
| **`protected`** | Visible in the same package and in child classes. |
| **`public`** | Visible everywhere. |
| **Serialisation** | Turning an object into text (such as JSON). |
| **Setter** | A public method that changes a private field, ideally after checking the value. |
| **Short-circuiting** | `&&` and `\|\|` stopping as soon as the answer is known. |
| **Static field** | One copy shared by the whole class. |
| **`throw`** | Stop immediately and report a problem. |
| **UML** | A standard notation for drawing software designs. |
| **Validation** | Checking that a value is acceptable before storing it. |

---

## 17. One-page cheat sheet

```java
package uk.ac.westminster.productsapi;           // which folder this class lives in

public class Product {

    public static final double VAT = 0.2;         // constant: static + final, CAPITALS
    private static int count = 0;                 // one shared copy  (UML: underlined)

    private final Long id;                        // private + never changes
    private String name;                          // private + may change
    private double price;
    private boolean inStock;
    private Supplier supplier;                    // HAS-A: field of another class type

    public Product(Long id, String name, double price) {   // full constructor
        this.id = id;
        setName(name);                            // reuse the setter's checks
        setPrice(price);
        count++;
    }

    public Long getId() { return id; }            // getter: get + Capitalised
    public String getName() { return name; }
    public double getPrice() { return price; }
    public boolean isInStock() { return inStock; }   // boolean getter: is

    public void setName(String name) {            // setter with validation
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name is required");
        }
        this.name = name;
    }

    public void setPrice(double price) {
        if (price < 0) {
            throw new IllegalArgumentException("Price cannot be negative");
        }
        this.price = price;
    }

    public static int getCount() { return count; }   // call as Product.getCount()
}
```

| Remember | |
| :-- | :-- |
| Private data, public behaviour | `-` fields, `+` methods in UML |
| `getX`, `setX`, `isX` for booleans | Exact spelling matters (Jackson reads getters) |
| No getter = field missing from JSON | No error is shown |
| Validate in setters; constructor calls setters | Rules written once |
| No setter for values that never change | Use `final` |
| Spring needs a no-arg constructor | `public X() {}`, but not with `final` fields |
| Overloading: same name, different parameters | Java picks by the arguments |
| `static` = one shared copy | Call through the class name |
| UML: `name : Type` | Constructors have no return type; static is underlined |
| Association: solid line, arrow at the held class | "has a" |

---

## 18. Am I ready for Phase 3?

- [ ] I can name the four access modifiers and their UML symbols. (§2)
- [ ] I can make a class's fields private and write correctly named getters, including `is` for booleans. (§3, §5)
- [ ] I can write a setter that rejects bad values with `IllegalArgumentException`. (§4, §6)
- [ ] I can make the constructor reuse the setter's validation. (§6.2)
- [ ] I can decide which fields deserve setters and which should be `final`. (§7)
- [ ] I can write a no-arg constructor and a full constructor in one class, and explain overloading. (§8)
- [ ] I can explain why a missing getter makes a field vanish from JSON. (§9)
- [ ] I can write a class with a field of another class type, and explain HAS-A. (§10)
- [ ] I can write and call a static field and a static method. (§11)
- [ ] I can draw a three-compartment UML box from code, and write code from a UML box. (§12)
- [ ] I built `BankAccount` without copying. (§15.3)

### The question that opens Phase 3

Lecture 3 begins where Tutorial 2 ends. The shop can return one product, but it sells exactly one, and it can never change:

```java
Product p1 = new Product(1L, "Laptop", 899.99);
Product p2 = new Product(2L, "Mouse", 24.99);
Product p3 = new Product(3L, "Monitor", 249.99);
```

A variable per product. The count is frozen in the code. Nothing can be added while the program runs. **How do you hold any number of objects, add and remove them, and find one?**

**Next: Phase 3, Collections and Generics.**
